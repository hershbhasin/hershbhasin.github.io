---

layout: post

title: "Azure VM Gold Images"

author: "Hersh Bhasin"
---

## Introduction

In this post, I discuss two approaches for providing pagination functionality to a CosmosDb  Telemetry API that follows the[ JSON API Spec  for Pagination](http://jsonapi.org/examples/#pagination), which specifies that a JSON API may provide "previous", "next" and "self" links in the response body that the user can follow to get the next/previous page sets.

```json
{
   "meta": {
    "total-pages": 10
    }, 
  "data": [{
    "type": "telemetry",
    "id": "1",
    "attributes": {
      "speed": 34.93,
      "longitude": -83.6948611,
      "latitude": 42.1621701,
      "fuel-consumed": 0.8205,
       "rpm" : 1103.90625,
       "odometer" : 2892
    }
    
  }], 
 "links": {
    "self": "http://abc.com/v1/vehicle/TST20160905US0001/",
    "prev": "http://abc.com/v1/vehicle/TST20160905US0001/?page=previouspage_state_encoded",
    "next": "http://abc.com/v1/vehicle/TST20160905US0001/?page=nextpage_state_encoded",
   
  }
  
}
```

The two approaches I consider are:

- Pagination using the CosmosDb provided Continuation Tokens
- Pagination using DateTime spans

The  "prev"  and "next" links above contain the encoded state of the "next" or "previous" page which is build on each API call: each  API request also  builds links for the next and previous pages, based on the strategy we choose. We would like to "silently" substitute the strategy if needed, hence the state is Base 64 encoded, so the user of our API is unaware of what the actual next/prev link looks like. When the user clicks on the prev/next link, the API decodes the appropriate state and navigates.

## Pagination using CosmosDb Continuation Tokens

**Building the "Next" link using Continuation Tokens**

- When defining the FeedOptions for a query, you restrict the page size (MaxItemCount) to a specific page size
- When the query executes, CosmosDb provides you with a "RequestContinuation" token for the next page set
- In your response, you  provide a "Next" link to to the user that contains the RequestContinuation token as a querystring
- In the next API request, you obtain the RequestContinuation from the querystring and set the FeedOptions with the token

```json
var spec = SqlQueries.Telemetry(parameters.Vin, parameters.PageSize, parameters.StartDate, parameters.EndDate);

//use the continuation token if you have it
FeedOptions feedOptions = new FeedOptions();
feedOptions.MaxItemCount = parameters.PageSize;
feedOptions.RequestContinuation = GetContinuationToken(parameters);


//query
var queryable  = (_client.CreateDocumentQuery(_defaultCollectionLink,
        spec,feedOptions)).AsDocumentQuery();

FeedResponse documents = await queryable.ExecuteNextAsync();

if (documents != null)
{
    requestList = documents.ToList();
    
    parameters.UpdateLastThreeTokens(documents.ResponseContinuation);
}
```

**Building the "Previous" link using Continuation Tokens** While the "Next" link  is simple to implement, as on each pass of the query, CosmosDb provides you with a "Next" continuation token, the "previous" link strategy is complicated by the necessity of storing the  continuation token with a specific page number. If from page 10, you want to go to page 9, you will have to use the continuation token for the (n-1) page, which in this example is the token for page 8. Building a "previous" link, with cumulative additions of token/page number combinations, as we navigate from page to page,  would soon become unwieldy.  This would  forces us to use a "stateful" strategy for pagination:  the continuation token, with the associated page number would have to be stored in some storage medium, and maintaining state  is not something we want in a stateless API. The use of continuation tokens is more appropriate for a client side solution and not for an API solution, **Pros & Cons of the Continuation Token  as the Pagination Strategy** **Pros**

- Native out of the box CosmosDb solution. Efficient if only "Next" is to be provided.

**Cons**

- Impractical if "Previous" has to be provided as you have to store the tokens and associated page numbers in a storage medium. Complex coding to implement "Previous"
- Tied in to a CosmosDb native solution. If we need to move away from CosmosDb, we will have to revisit the pagination strategy.


## Pagination using DateTime spans

This strategy uses the record timestamp  to build the "next" and "previous" links.  **Building the "Next" link  using DateTime span** The data comes back in the descending order of timestamp.  Using the specified "Page Size" (which becomes the Top n rows in the DocDb SQL query,)  you get the next page load, using the "Beginning-Of-Time" as the start date and the Last Record of current page set as the end date  (which is the minimum timestamp as the data is sorted descending) .**Building the "Previous" link  using DateTime span** The logic is reversed in the "Previous" strategy.  To go back to a previous page set, you  use the timestamp of the first record (max timestamp) of current page set as the start date and the "End-Of-Time" as the end date and you sort the query Ascending. That way the Top n pages of the resultant dataset is the "Previous" page set you need. In code, you sort the dataset descending and return it. **Example Implementation**

```json
public  SqlQuerySpec TelemetrySql(MethodParams.Telemetry parameters)
    {

        var queryParams = new SqlParameterCollection()
        {
            new SqlParameter("@vin", parameters.Vin),
            new SqlParameter("@pageSize", parameters.PageSize),
            new SqlParameter("@startdate", parameters.StartTime.ToDateFromString()),
            new SqlParameter("@enddate", parameters.EndTime.ToDateFromString())

        };

        string BaseSql = @"
            SELECT Top @pageSize
            doc.id,
            doc.timestamp,
            doc.outsideUseData.ODO,
            doc.outsideUseData.SP1,  
            doc.outsideUseData.NE1,
            doc.outsideUseData.FUGAGE,
            doc.location.coordinates[0] as longitude ,
            doc.location.coordinates[1] as latitude,
            doc.outsideUseData.B_FC
            FROM doc
            WHERE (doc.vin = @vin)
           
        ";

        //Default Period
        string PeriodSql = @"
            AND doc.timestamp > @startdate
            AND doc.timestamp <= @enddate
         ";

        //Default Ordr By
        string OrderBy = @"
         ORDER BY doc.timestamp DESC
        ";

        //Next
        if (parameters.Direction == DirectionEnum.Next)
        {

            PeriodSql = @"
            AND doc.timestamp > @startdate
            AND doc.timestamp < @enddate
            ";
        }

        //Previous
        if (parameters.Direction == DirectionEnum.Previous)
        {

            PeriodSql = @"
             AND doc.timestamp > @startdate
             AND doc.timestamp < @enddate
             ";
            OrderBy = @" ORDER BY doc.timestamp";
        }


        //Return Spec

        var spec = new SqlQuerySpec();
        spec.QueryText = string.Concat(BaseSql, PeriodSql, OrderBy);
        spec.Parameters = queryParams;



        return spec;
}
```

 

**Pros of using DateTime spans**

- Stateless solution.
- Platform Agnostic: It is not bound to CosmosDb. If we need to move to another storage solution (like Hbase etc.) this approach will still work

**Cons of using DateTime spans**

- Custom solution.  Slightly complex logic for  implementing "previous" .
- Query degrades as larger duration/ time span is selected.

## Conclusion

CosmosDb does not have a good stateless solution for implementing a "Previous" recordset functionality. To revisit a previous page, the associated continuation token for that page is required to be known. This is not very practical for an API, which is best kept stateless. The client side application can and should maintain state. One solution could be that on each API call, the API payload returns the continuation token to the client, who stores it and sends it back as necessary to the API. However, this is a client specific solution. Another drawback of using continuation tokens is that one is bound to a CosmosDb specific solution. If we need to move to another platform, pagination using CosmosDb continuation tokens will not work. For these reasons, we choose to implement pagination using the DateTime spans approach as described in this article.      