---
author: Liam Björkman
tags: azure serverless containers
---

## What is Serverless and Function as a Service (Faas)?



### Calculator Program

The following is a very simple calculator that only knows how to calculate the sum of two numbers that are supplied by using a HTTP GET query.

```c#
public static class HttpCalcTrigger
    {
        [FunctionName("HttpCalcTrigger")]
        public static IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req, ILogger log)
        {
            string[] numbers = req.Query["numbers"].ToString().Split(' ');
            try
            {
                int n1 = int.Parse(numbers[0]);
                int n2 = int.Parse(numbers[1]);
                int result = n1 + n2;
                log.LogInformation($"C# HTTP trigger function processed {n1} + {n2} and returned {result} .");
                return new OkObjectResult(result);
            }
            catch
            {
                return new BadRequestObjectResult("Numbers were incorrectly supplied.");
            }
        }
    }
```

#### Bronze Level

The program has been created using the Azure Function template from the Visual Studio Code Azure Extension. The function is configured to be triggered on any HTTP request that has a valid API key supplied as seen above in the method signature `HttpTrigger(AuthorizationLevel.Function)`

![image-20210915160152621](C:\Users\nissi\AppData\Roaming\Typora\typora-user-images\image-20210915160152621.png)

Using VS we also have access to debug the function locally, so we can set breakpoints just like in a normal application, using this we can access the application in our browser on a local IP, under is an example of how we run the program locally.

![image-20210915160528733](C:\Users\nissi\AppData\Roaming\Typora\typora-user-images\image-20210915160528733.png)

​																							Our app running locally with `numbers=50+50` as query

We also can see any log output in our integrated terminal in VS Code

![image-20210915160750241](C:\Users\nissi\AppData\Roaming\Typora\typora-user-images\image-20210915160750241.png)



From the extension we can directly deploy our function to Azure, after is has finished we can access it using the azure URI. Under is the same application but running in the cloud instead. 

![image-20210915161428771](C:\Users\nissi\AppData\Roaming\Typora\typora-user-images\image-20210915161428771.png)

To run this in the cloud we also have to supply our API key, as specified in the `code` part of the query

We can see logs from the Azure portal in the same way as we could in our local terminal.

![image-20210915161543882](C:\Users\nissi\AppData\Roaming\Typora\typora-user-images\image-20210915161543882.png)



### Silver Level

