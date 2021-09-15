---
author: Liam Björkman
tags: azure serverless containers
---

## What is Serverless and Function as a Service (Faas)?

TODO

## Calculator Program

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

### Bronze Level

The program has been created using the Azure Function template from the Visual Studio Code Azure Extension. The function is configured to be triggered on any HTTP request that has a valid API key supplied as seen above in the method signature `HttpTrigger(AuthorizationLevel.Function)`

<img src="/img/image-20210915160152621.png">

Using VS we also have access to debug the function locally, so we can set breakpoints just like in a normal application, using this we can access the application in our browser on a local IP, under is an example of how we run the program locally.

<img src="/img/image-20210915160528733.png">

Our app running locally with `numbers=50+50` as query



We also can see any log output in our integrated terminal in VS Code

<img src="/img/image-20210915160750241.png">



From the extension we can directly deploy our function to Azure, after is has finished we can access it using the azure URI. Under is the same application but running in the cloud instead. 

<img src="/img/image-20210915161428771.png">

To run this in the cloud we also have to supply our API key, as specified in the `code` part of the query

We can see logs from the Azure portal in the same way as we could in our local terminal.

<img src="/img/image-20210915161543882.png">



### Silver Level

We can automate this process to create the same setup directly from the Azure CLI using a bash script. 

<img src="/img/azurecli1.png">

The following script clones our existing calculator app from GitHub and creates all Azure Resources that we previously did manually.

```bash
#!/bin/bash

# Function app and storage account names must be unique.
storageName=asduwqdgy2237$RANDOM
functionAppName=bjork-dev-github$RANDOM
region=northeurope

# TODO:
gitrepo=https://github.com/bjork-dev/AzureFunctions.git
token=removedFromThisPublicScript

# Enable authenticated git deployment in your subscription from a private repo.
az functionapp deployment source update-token \
  --git-token $token

# Create a resource group.
az group create \
  --name myResourceGroup \
  --location $region

# Create an Azure storage account in the resource group.
az storage account create \
  --name $storageName \
  --location $region \
  --resource-group myResourceGroup \
  --sku Standard_LRS

# Create a function app with source files deployed from the specified GitHub repo.
az functionapp create \
  --name $functionAppName \
  --storage-account $storageName \
  --consumption-plan-location $region \
  --resource-group myResourceGroup \
  --deployment-source-url $gitrepo \
  --deployment-source-branch master \
  --functions-version 2

```

We can see the output from the CLI while the script is running

<img src="/img/azurecli2.png">
