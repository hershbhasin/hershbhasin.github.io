---

layout: post

title: "Azure VM Gold Images"

author: "Hersh Bhasin"
---

## Introduction

In a recent engagement I had to devise a way to secure a Json Api that surfaced data from a Azure CosmosDb collection. The use case was that a API consumer should first be *authenticated* by suitable means and once his/her identity was established, *authorized* to only query a subset of data to which he/she had rights to. This right was established by an array of unique record keys that the consumer was allowed to see. Since the data that the API surfaced was car telemetry data, and the unique key was vehicle VIN, the user was to be restricted to a set of VINs.

## Authorization Approach

Azure Api Management (APIM) was used as the authorization mechanism. I will discuss the detailed APIM approach later in this post, but in brief, APIM allows one to publish APIs to which the users subscribe via the APIM developer portal. When they subscribe, they are issued a subscription token and the user passes this token in the header of each API call.

With APIM policies it is possible to extract the users subscribed email, by looking up the subscription token he/she supplies. As I explain later, this email will be used for authorization purposes (what VINS does the user have rights to based on his rights, given that the user is identified by his email?) I could have used the subscription token itself as the lookup key, however a user can regenerate his subscription key. The email is more immutable.

## Authentication Approaches Considered

I considered the following approaches:

**App: Service Logic**

- The app service logic is responsible for enforcing authorization.
- Store authorized vehicle list for each customer.
- Explicitly apply filter to query & constrain results to authorized vehicle VINs only.

**CosmosDb: By Partition Key**

- The collection is partitioned by VIN. Partition permissions are granted for each customer based on authorized vehicles.
- Store authorized vehicle list for each customer.
- Grant/revoke user-specific permissions for each authorized partition key (VIN).
- Utilize user-specific permission tokens when querying DocumentDB to enforce authorization policies.

**CosmosDb: By Collection**

- A collection is created for each customer. Customer access is limited to the authorized collection.
- Store authorized vehicle list for each customer.
- Store vehicle data in the appropriate collection (potentially multiple/duplicate) based on authorization list.
- App service will need to query the appropriate collection based on the customer. Could be facilitated by collection authorization keys.

##  Choice of Authentication Approach

I found the first approach App:Service logic to be most flexible. I created an "Auth" CosmosDb collection which had a json document per user that had an array of VINS that he had access to. The document looked like this:

```json
{ 
   "Vins" : [ 
    "TEST-VXFA50-609060",
    "TEST-VXFA50-609061" 
   ], 
   "id" :  "first_client@go.com" 
} 
```

The "id" is the email of the user which is extracted by a Policy on the APIM and passed in the header of the request by APIM. This is discussed in the Authorization section below. The API extracts the email from the header and makes a query to CosmosDb "Auth" collection and gets the list of VINs the user identified by the email has access to, and matches it with the VIN that was requested by the user in the current API request. This is how authentication was established.

##  Authorization Approach

Azure API Management was used to publish APIs. APIs are published using the Publisher Portal and once they are published, consumers of the API subscribe to the APIs via a developer portal. As part of the registration process, they provide a email. They obtain a subscription token at the developer portal, which they pass in the header of each API call.

An APIM Policy can be applied to the API using the Publisher portal that can intercept and transform the request or the response of the API. I used a APIM Policy to extract the email of the user and pass it in the subsequent header of the API request (the request-email header). I also added a check in Policy so that a user could not spoof the system by manually adding a header called request-header.

In the policy below, I am checking for a header called request-email and if it exists, someone is trying to hack my API so I return a 400 Bad Request , else I set the request-email header with the user email.

```json
<policies>
    <inbound>
        <choose>
            <when condition="@(context.Request.Headers.GetValueOrDefault("request-email","").Length >0)">
                <return-response response-variable-name="existing response variable">
                    <set-status code="400" reason="Bad Request" />
                    <set-header name="Bad-Request" exists-action="override">
                        <value>error="Bad request"</value>
                    </set-header>
                </return-response>
            </when>
        </choose>
        <set-header name="request-email" exists-action="override">
            <value>@(context.User.Email)</value>
        </set-header>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
</policies>
```