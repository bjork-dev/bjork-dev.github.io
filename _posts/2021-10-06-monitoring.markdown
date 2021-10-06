---
author: Liam Björkman
tags: cloud files blobs azure storage
---

## The Application

I've extended my Todo application from my Web Applications post. More info on it can be found here.



## Diagram

The following diagram show how the services are connected.

<img src="/img/s.png">

There's also a graph function in Applications Insights called [Application Map](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-map?tabs=net), that gives you an overview over calls made between Azure connected systems. The app map finds components by following HTTP dependency calls made between servers with the Application Insights SDK installed.

<img src="/img/azuremap.png">



## Code

This will only cover the logging code part, all other code can be found in previous post.

### Startup.cs

To enable usage of server-side telemetry I've added the service provider and specified my instrument key to be retrieved from Configuration (`appsettings`).

```c#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddApplicationInsightsTelemetry(Configuration);
            ....
        }
```

### _ViewImports.cshtml

To also inlide client-side telemtry which gives access to addintional data collection methods, I've first injected the JavaScript into my app.

```c#
@inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet JavaScriptSnippet
```

### _Layout.cshtml

Then in layout added it in the head section, which will render on each page.

```html
 ...
 @Html.Raw(JavaScriptSnippet.FullScript)
</head>
```

That's all that needs to be done in order to start using Application Insights SDK. After deployment with new code changes we can quickly check that the telemetry is working by going to 'Live Metrics' on our AI page. If it's up and running we should se data in real-time.

<img src="/img/livemetrics.png">

### Queries

We can use queries powered by [Kusto](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/) to retrieve data from our Azure resources. Under is 2 samples of such queries.

#### Operations Performance

This query allow us to seehow long our HTTP requests to our page takes.

```sql
requests
| summarize RequestsCount=sum(itemCount), AverageDuration=avg(duration), 
	percentiles(duration, 50, 95, 99) by operation_Name
| order by RequestsCount desc // order from highest to lower (descending)
```

Produces the following output

<img src="/img/ai1.png">

We can with the following data see that our Index page takes an average of 1.8 seconds to load, and 95th percentile is almost 6 seconds. In a real-world scenario, this could be valuable information to investigate further.

#### Top 10 countries by traffic

This query allow us to see where the most traffic is coming from. Using this info we could strategically scale out our application to datacenters near our customers.

```sql
requests
| summarize CountByCountry=count() by client_CountryOrRegion
| top 10 by CountByCountry
| render piechart
```

Produces the following output, with a uncategorized country for unknown reasons. Could be our other Azure resources that are making these requests.

<img src="/img/ai2.png">

### Using Applications Insights for increased security 
