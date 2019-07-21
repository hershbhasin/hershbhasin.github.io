---
layout: post
title: "AZ 300 Active Directory Study Notes"
author: "Hersh Bhasin"
comments: true
categories: AZ-300 Azure-Active-Directory
published: true

---

# Introduction

These are notes for AZ-300 prep for Azure Active Directory.  I am following [Pluralsight Course by John Savill](https://app.pluralsight.com/library/courses/microsoft-azure-managing-active-directory/table-of-contents) and  [eDX course: Microsoft Azure Identity](https://courses.edx.org/courses/course-v1:Microsoft+AZURE204x+1T2019a/course/)

## Authentication Commonly Used 

**Authorization**: Proving who I am.

**Authentication**: What I am allowed to do.

**OAuth2**:  Idea of consent: I want to give this particular application or service to act on my behalf in a certain scope.

In cloud authentication and authorization is quite often merged because in the cloud authorization is often used as a pseudo authentication i.e. if I have been authorized to do something, I can assume that I have  been authenticated. 

For example Oauth2 , the most common type of authentication used by Azure AD is strictly authorization, not authentication: a owner of a recourse (a user,)  gives consent to some client service (an application), to access some resources he owns ( a scope) and perform certain actions on his behalf.  *The service actually got authorization to delegate as that user, but we can assume that because the service got this authorization, it must have been authenticated.*

**OpenID Connect**: Sits on top of OAuth2 and is there strictly to provide authentication: to prove the user is who he says he is. It uses a JWT token

**SAML/WS-FED**: WS-Fed is more Microsoft-specific. SAML is an open-based standard for exchanging authentication and authorization data between parties. SAML is what we are commonly going to use with Federation. If we are using ADFS, behind the scenes we're using SAML as the format for the assertions we are using, the claims about the user

**Utilize MFA when possible**

#  Azure Active Directory Service

Its called Active Directory but is not Active Directory. It is an identity provider geared towards the cloud

**Azure AD Domain Services**: Enables limited machine membership and policy application for Azure services where **legacy** authentication protocol (Kerbros, NTLM, maybe bind with LDAP or ADSI) and binding support is required.

However regular AAD is not speaking legacy protocols like Kerbros, which is simply not a good fit for internet based service where communication is generally limited to HTTPS.

 **Azure AD ad min center** : https://aad.portal.azure.com

**Azure Shell**:  https://shell.azure.com

![ad1](..\assets\ad1.PNG)



![ad1](..\assets\ad2.PNG)



[Install and use the log analytics views for Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/reports-monitoring/howto-install-use-log-analytics-views)

## RBAC

Applied to:

* Management Group

* Subscription

* Resource Group

* Resource

![ad1](..\assets\ad3.PNG)

![ad1](..\assets\ad4.PNG)

## Roles

1. Azure AD Roles: for controlling access to  cloud applications that trust ad (SharePoint, Skype, Intune etc.) Accessed through Active Directory/Roles & Administrators
2. Resource Manager Roles:  for controlling access to Azure resources. Accessed through Subscriptions/Access Control/Roles. Role permissions are based on "Resource Providers". A role might have a number of Resource Providers (e.g. Microsoft Compute, Microsoft Storage etc). Roles are applied to a scope:  Management Groups, Subscription, RG, Resource etc.
   1. Owner
   2. Contributor
   3. Reader

![ad1](..\assets\ad7.PNG)

![ad1](..\assets\ad5.PNG)

![ad1](..\assets\ad6.PNG)

[Video: Azure AD Pass Through and Seamless Sign In](https://courses.edx.org/courses/course-v1:Microsoft+AZURE204x+1T2019a/courseware/e670c9a7-9dd2-fc53-cfbd-cba717cbb879/90ffbe48-7302-f5b1-2748-0a1b4382f6fa/3?activate_block_id=block-v1%3AMicrosoft%2BAZURE204x%2B1T2019a%2Btype%40vertical%2Bblock%40993020da-5aea-8d23-3367-9dd510b8d124)

![ad1](..\assets\ad8.PNG)

![ad1](..\assets\ad9.PNG)

![ad1](..\assets\ad10.PNG)

![ad12](..\assets\ad12.PNG)

![ad12](..\assets\ad13.PNG)

Note: with synchronization,  Azure AD contains passwords, synchronized from on-prem AD. However in pass-through, passwords remain on-prem and Azure AD has no passwords. On login,  on-prem AD is asked to authenticate credentials.

Azure AD Connect with Pass Through gives similar results as Federation, with much simple setup.

**Azure AD Connect Health** 

Azure AD Connect Health is an Azure AD Premium feature (Connect is available in Free and Basic, but Health requires Premium) that will monitor on-premises AD DS identities and provide alerts. This requires an agent on each server being monitored.

![ad13](..\assets\ad13a.PNG)

![ad13](..\assets\ad14.PNG)

![ad13](..\assets\ad15.PNG)

## Tenant

A tenant is simply a dedicated instance of Azure AD that your organization receives and owns when it signs up for a Microsoft cloud service such as Azure or Office 365. For example, contosogold.onmicrosoft.com, is a tenant.

A tenant houses the users in a company and the information about them - their passwords, user profile data, permissions, and so on. It also contains groups, applications, and other information pertaining to an organization and its security.

You can have multiple tenants within your organization. Each tenant can have a different purpose and fulfill a different scenario. For example, you might have tenant for Testing, Office365, and Production.

Can you think of reasons why you might want different tenants?

- **Isolation**. Each tenant is isolated with different policies, users, groups, and roles.
- **Resources**. Each tenant can have different resources specific for their functionality.
- **Administration**. Each tenant can have different administrator roles.
- **Synchronization**. Each tenant can implement synchronization in a different way.

To use a tenant, it must be associated with a subscription. The basic steps are: create a directory, create an admin for the directory, and then have the admin associate the directory with a subscription. Each directory must have at least one subscription.

![](..\assets\ad16.PNG)

![](..\assets\ad17.PNG)

**Tenant vs Subscription**

A subscription is a credit card definition: the entity that pays for using Azure resources. A Tenant is an instance of Azure Active directory

![](..\assets\ad18.PNG)

![](..\assets\ad19.PNG)

Guest users are user added to Azure AD from a third party like Microsoft or Google.

![](..\_posts\ad20.PNG)



![ad21](..\assets\ad21.PNG)

![ad21](..\assets\ad22.PNG)

![ad21](..\assets\ad23.PNG)

![ad21](..\assets\ad24.PNG)

![ad21](..\assets\ad25.PNG)

![ad21](..\assets\ad26.PNG)

![ad21](..\assets\ad27.PNG)

![ad21](..\assets\ad28.PNG)

![ad21](..\assets\ad29.PNG)