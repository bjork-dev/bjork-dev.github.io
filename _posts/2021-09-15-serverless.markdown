---
author: Liam Björkman
tags: azure serverless containers FaaS
---

## What is Serverless and Function as a Service (Faas)?

### Serverless

Serverless means that the cloud provider takes care of the complete infrastructure so that the developer can focus solely on the code. Meaning that there's no configuration, you just deploy your code to the provider and it's up and running. 

The name may be misleading, there are still servers in the background, the difference is that the infrastructure layer has been abstracted from the customer.

### Function as a Service

FaaS is a type of cloud-computing service that allows execution of code in response to events such as a HTTP trigger.  It is actually a subset of Serverless, but it is the most central technology in Serverless architecture, therefore they easily get mixed up. One key aspect of FaaS is the automatic management of resource allocation (scaling up/down) and the billing strategy. FaaS computing architecture never holds compute resources in memory, instead computing takes place only when requested to (for example when a trigger is hit). This results in a efficient price to performance ratio, since you only pay for the compute time.

## Calculator Program

The following is a very simple calculator that only knows how to calculate the sum of two numbers that are supplied by using a HTTP GET query.



Run.csx - The core application that will be run as a function.

```c#
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    string[] numbers = req.Query["numbers"].ToString().Split(' '); // The '+' sign dissapears in the request, therefore we split on whitespace instead. Not ideal in a real app.
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

```

function.json - Describes the input and output flow, as well as what HTTP methods to accept.

```json
{ 
    "bindings": [
        {
          "authLevel": "function",
          "name": "req",
          "type": "httpTrigger",
          "direction": "in",
          "methods": [
            "get"
          ]
        },
        {
          "name": "$return",
          "type": "http",
          "direction": "out"
        }
      ]
}
```



### Bronze Level

The program has been created using the Azure Function template from the Visual Studio Code Azure Extension. The function is configured to be triggered on any HTTP request that has a valid API key supplied as seen above in the method signature `HttpTrigger(AuthorizationLevel.Function)`.

<img src="/img/image-20210915160152621.png">

Using VS we also have access to debug the function locally, so we can set breakpoints just like in a normal application, using this we can access the application in our browser on a local IP, under is an example of how we run the program locally.

<img src="/img/image-20210915160528733.png">

Our app running locally with `numbers=50+50` as query



We also can see any log output in our integrated terminal in VS Code.

<img src="/img/image-20210915160750241.png">



From the extension we can directly deploy our function to Azure, after is has finished we can access it using the azure URI. Under is the same application but running in the cloud instead. 

<img src="/img/image-20210915161428771.png">

To run this in the cloud we also have to supply our API key, as specified in the `code` part of the query.

We can see logs from the Azure portal in the same way as we could in our local terminal.

<img src="/img/image-20210915161543882.png">



### Silver Level

We can automate this process to create the same setup directly from the Azure CLI using a bash script. 

<img src="/img/azurecli1.png">

The following script clones our existing calculator app from GitHub and creates all Azure Resources that we previously did manually.

```bash
#!/bin/bash

# Function app and storage account names must be unique.
storageName=bjork-dev-storage$RANDOM
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
  --deployment-source-branch main \
  --functions-version 2

```

We can see the output from the CLI while the script is running.

<img src="/img/azurecli2.png">

After the script has finished running we can verify that the Function App & HttpTrigger has been created by checking the Portal.

<img src="/img/azureportal1.png">

Since we have set authentication on our trigger to avoid usage by unauthorzied users we have to supply our API key when running the trigger in the browser.

<img src="/img/image-20210916105505656.png">

### Gold Level

To be able to automate this process through a pipeline we have to setup a repository secret to be able to authenticate to Azure automatically.

To do this we first have to have an existing function app, the deployment is only for functions. We then can get the publish profile from the app menu.

<img src="/img/calc1.png">

We will then create the secret in our repo using the XML data. Using this procedure instead of adding it as a textfile to the repo hides our sensitive data from public eyes. The data is collected at pipeline runtime by environment variables instead.

<img src="/img/gh2.png">

The layout in the repository will look different when using this path. Instead of `*.csx` files we will have normal `*.cs` files inside a local function app folder, each `*.cs` file will be a separate trigger, and all can be deployed at once to the function app in Azure.

<img src="/img/vscode1.png">

The code is more or less the same, except that the namespace is required. Notice also that the HTTP methods are now defined in the method signature instead, and `function.json` has been removed from the project. This template supersedes the old `*.csx` way of creating functions.

```c#
namespace LocalFunctionApp
{
    public static class HttpTrigger
    {
        [FunctionName("HttpTrigger")] // We can set custom names for Azure here
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
            ILogger log)
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
}
```

Our workflow file looks like this.

```yaml
name: Deploy DotNet project to Azure Function App

on:
  [push]

# CONFIGURATION
# 1. Set up the following secrets in your repository:
#   AZURE_FUNCTIONAPP_PUBLISH_PROFILE
#
# 2. Change these variables for your configuration:
env:
  AZURE_FUNCTIONAPP_NAME: calculator-function-app  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: './LocalFunctionApp' # The folder containing all our triggers
  DOTNET_VERSION: '3.1.x'              # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    environment: dev
    steps:
    - name: 'Checkout GitHub Action'
      uses: actions/checkout@master

    - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: 'Resolve Project Dependencies Using Dotnet'
      shell: pwsh
      run: |
        pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
        dotnet build --configuration Release --output ./output
        popd
    - name: 'Run Azure Functions Action'
      uses: Azure/functions-action@v1
      id: fa
      with:
        app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
        package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
        publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
```

The flow is similar to previous ones we've created, the real difference is the last step where the deployment part takes place. The azure function action is run which gets our Azure Function App name and secrets and then takes care of the rest, pushing the code to a new trigger in the app repo in Azure.

The deployment part takes less than a minute and we can follow it in the Actions tab.

<img src="/img/action1.png">

After completion, we can verify in the portal that the trigger has been created, and run it from the URI.

<img src="/img/azureportal2.png">



### Security

We've taken several security precautions to make sure that our application is not exposed to any malicious actors. Since we have a public repository, hardcoding variables such as login information is a very bad idea, instead we've used secrets and environment variables to abstract this part from the code and let the pipeline take care of it at runtime. 

To secure our actual application we've opted to require an API key to even reach the app, which must be supplied at every request. Since our app is very basic and does not use any external systems, such as databases, etc. there's not really any vulnerabilities. But to validate input and make sure no unexcepted exceptions are thrown, we've wrapped the logic in a try-catch clause with a fixed error message and also run `int.Parse` to make sure that the input is a number.

##### Sources

##### [Know About Serverless](https://geekflare.com/know-about-serverless/)

##### [FaaS - IBM](https://www.ibm.com/cloud/learn/faas)

##### [Build Serverless APIs with Azure Functions](https://channel9.msdn.com/Shows/Azure-Friday/Build-serverless-APIs-with-Azure-Functions)

##### [Go serverless: Event-driven applications with Azure Functions](https://www.youtube.com/watch?v=UFxQhszT450)

##### [Using GitHub Actions With Azure Functions](https://www.aaron-powell.com/posts/2020-02-28-using-github-actions-with-azure-functions/)

##### [OWASP Top 10 (2017) Interpretation for Serverless](https://raw.githubusercontent.com/OWASP/Serverless-Top-10-Project/master/OWASP-Top-10-Serverless-Interpretation-en.pdf)

