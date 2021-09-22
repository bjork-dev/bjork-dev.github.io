---
author: Liam Björkman
tags: azure serverless containers FaaS
---

The following describes a complete serverless system consisting of and CRUD API and a Azure Cosmos database.



### The Application - Todo API

Our function app consists of the following HHTP triggers that describes an API for managing todos.

* AddTodo [POST] - Adds a new todo to the database from the request body.
* DeleteTodo [DELETE] - Deletes the todo specified by query ID.
* EditTodo [PUT] - Updates the todo as specified by the query ID with the request body.
* GetTodo [GET] - Gets all todos.
* GetTodoById [GET] - Gets a todo as specified by the query ID.
* Azure Cosmos DB - Our serverless database.



### Code

All triggers are pretty similar, a few key points to know about before reading the code blocks under are:

**The CosmosDB attribute** - Thanks to this relatively new attribute, we don't have to hardcode our connection to the database, instead the attribute collects relevant information and establishes the connection on execution. The attribute needs to bind to a variable, for example in the `AddTodo` function we bind the connection to `Todo document`, meaning that any data manipulation done to the object will be interpreted by `CosmosDB`. In the `AddTodo` example we create a new document. Basically all database actions are abstracted from the developer.

**DocumentClient class** - This is used for deleting and updating a document, I tried to achieve this with the attribute since we can use that to `POST` and `GET`. Unfortunately I did not get anywhere with this and the documentation for the `CosmosDB` attribute is non-existent. So instead I opted to use the `DocumentClient` class to perform these actions, which only adds about 2 lines of code anyway.

**Enviroment Variables** - These are located in our `local.settings.json`, and in `settings.json` when deployed to Azure. We can upload and download settings directly to our function app from VS code via the Azure extension.

<img src="/img/azfunc1.png">

The settings are shared by all our triggers and so is our model, so we only have to define these once.

#### Todo Model

Our model for databinding. Notice the `JsonProperty` attributes, these are needed for deserialization.

```c#
public class Todo
    {
        private string title;
        [JsonProperty("id")]
        public string Id { get; set; }
        [JsonProperty("title")]
        public string Title { get; set; }
        [JsonProperty("description")]
        public string Description { get; set; }
        [JsonProperty("dueBy")]
        public string DueBy {get; set;}
    }
```



#### AddTodo

```c#
[FunctionName("AddTodo")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = "todo")] HttpRequest req,
             [CosmosDB(
                databaseName: "TodoDb",
                collectionName: "TodoItems",
                ConnectionStringSetting = "CosmosDbConnectionString")]
                out Todo document,
                ILogger log)
        {
            try
            {
                var todo = new StreamReader(req.Body).ReadToEnd();
                document = JsonConvert.DeserializeObject<Todo>(todo);

                log.LogInformation($"C# HTTP trigger function inserted one row");

                return new OkObjectResult(document);
            } catch (Exception e)
            {
                document = null;
                return new BadRequestObjectResult(e.Message);
            }
        }
```



#### GetTodo

```c#
[FunctionName("GetTodo")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = "todo")] HttpRequest req,
            [CosmosDB(
                databaseName: "TodoDb",
                collectionName: "TodoItems",
                ConnectionStringSetting = "CosmosDbConnectionString")]
                IEnumerable<Todo> todoItems,
                ILogger log)
        {
            if(todoItems == null)
                return new NotFoundObjectResult("No entries found.");
            return new OkObjectResult(todoItems);
        }
```

#### GetTodoById

```c#
 [FunctionName("GetTodoById")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = "todo/{id}")] HttpRequest req, // Route is used for getting the value
            [CosmosDB(
                databaseName: "TodoDb",
                collectionName: "TodoItems",
                ConnectionStringSetting = "CosmosDbConnectionString",
                PartitionKey = "{id}", // Used for querying the table
                Id = "{id}")] Todo todoItem, // Used for binding to the class
            ILogger log)
        {
            if (todoItem == null)
            {
                log.LogError($"Todo item not found");
                return new NotFoundObjectResult("Todo item not found");
            }
            return new OkObjectResult(todoItem);
        }
```



#### EditTodo

```c#
private static DocumentClient client;
        
private static readonly string _endpoint = Environment.GetEnvironmentVariable("AccountEndpoint", EnvironmentVariableTarget.Process);

private static readonly string _key = Environment.GetEnvironmentVariable("AccountKey", EnvironmentVariableTarget.Process);

        [FunctionName("EditTodo")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "put", Route = "todo/{id}")] HttpRequest req,
             [CosmosDB(
                databaseName: "TodoDb",
                collectionName: "TodoItems",
                PartitionKey = "{id}",
                Id = "{id}",
                ConnectionStringSetting = "CosmosDbConnectionString")]
                Todo todo,
                ILogger log)
        {
            try
            {
                string id = todo.Id;
                var obj = new StreamReader(req.Body).ReadToEnd();
                todo = JsonConvert.DeserializeObject<Todo>(obj);
                todo.Id = id; // By assining here, we don't need to include ID in the request body.
                client = new DocumentClient(new Uri(_endpoint), _key);
                await client.ReplaceDocumentAsync(UriFactory.CreateDocumentUri("TodoDb", "TodoItems", todo.Id), todo, new RequestOptions { PartitionKey = new Microsoft.Azure.Documents.PartitionKey(todo.Id) });

                log.LogInformation($"C# HTTP trigger function inserted one row");

                return new OkObjectResult(todo);
            } catch (Exception e)
            {
                return new BadRequestObjectResult(e.Message);
            }
        }
```

#### DeleteTodo

```c#
private static DocumentClient client;

private static readonly string _endpoint = Environment.GetEnvironmentVariable("AccountEndpoint", EnvironmentVariableTarget.Process);

private static readonly string _key = Environment.GetEnvironmentVariable("AccountKey", EnvironmentVariableTarget.Process);

        [FunctionName("DeleteTodo")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "todo/{id}")] HttpRequest req,
            [CosmosDB(
                databaseName: "TodoDb",
                collectionName: "TodoItems",
                PartitionKey = "{id}",
                Id = "{id}",
                ConnectionStringSetting = "CosmosDbConnectionString")] Todo todo,
            ILogger log)
        {
            try
            {
                client = new DocumentClient(new Uri(_endpoint), _key);
                await client.DeleteDocumentAsync(UriFactory.CreateDocumentUri("TodoDb", "TodoItems", todo.Id), new RequestOptions { PartitionKey = new Microsoft.Azure.Documents.PartitionKey(todo.Id) });
                return new OkObjectResult("Deleted " + todo.Title);
            }
            catch(Exception e)
            {
                return new NotFoundObjectResult(e.Message);
            }
        }
```



### Database

The database server is called `todo-server` and has one database called `TodoDb` with one collection called `TodoItems`.

The scheme looks like this.

```json
{

 "id": "4dd4c474-cdde-423a-8107-c965b4550640",

 "title": "Sample Todo",

 "description": "Blablablabla",

 "dueBy": "2021-09-22"

}
```

The partition key for the collection is named `/Id`, but there's also an id attribute in the json for usage in `GET`, etc. The partition key is only for indexing by the database engine.



### Deployment

The functions are hosted in GitHub. From there we can deploy through a pipeline to our function app in Azure.

The following describes the worklow.

```yaml
name: Deploy DotNet project to Azure Function App

on:
  [push]

# CONFIGURATION
# For help, go to https://github.com/Azure/Actions
#
# 1. Set up the following secrets in your repository:
#   AZURE_FUNCTIONAPP_PUBLISH_PROFILE
#
# 2. Change these variables for your configuration:
env:
  AZURE_FUNCTIONAPP_NAME: todo-github  # set this to your application's name
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'    # set this to the path to your web app project, defaults to the repository root
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

As described in the comment and in my previous post, a Azure Publish Profile secret must first be added to the repository.

After that the workflow is more or less the same as the previous one.

### Handling model updates

Cosmos DB is a schema-less NoSQL database. Each document is completely independent from each other and it doesn't require you to change  anything in order to add a new property. You can go straight to the portal and do it or programmatically through an SDK or a stored procedure.

To update existing documents you would need to write some kind of function that adds the new property and value to each document.

### Cost

Our database is based on the **Provisioned Throughput** model. It is measured in request units per second (RU/s) and billed per hour.

**R/U** - *Request units* - A `GET` request equals one R/U, but more heavy actions might require several R/Us.

Pricing model from Azure, a minimum of 400 R/Us must be reserved. We will base our pricing on the the **Single-Region write account.**

<img src="/img/az.png">

#### Almost no users scenario

This would probably be covered by 400 R/Us per hour which would land at around **200 kr** per month.

Azure functions are free to for the first 1,000,000 executions.  But if we base it at 2,000,000 executions, 128MB memory size & each exection takes 1000ms that leaves us at **1.75 kr** per month.

**Total: 202kr**

#### A lot of users scenario

For this scenario we would probably want multiple regions as well, but for simplicity sake we will use one.

Let's assume that we have 5000 users every hour making requests. Some of these requests are heavy (running stored procedure and joining queries). So we should probably reserve **15000 R/Us** just to be safe. That would cost us around **7650 kr** per month.

For the functions we might want around 7,000,000 executions per month to cover our expected user requests + some overhead. We would probably want to up the memory size to 1024MB as well. This would cost us **933 kr** per month based on 1000ms execution time.

**Total: 8,583 kr**



##### Sources

##### [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

##### [CosmosDB Pricing](https://azure.microsoft.com/en-us/pricing/details/cosmos-db/)

##### [Azure Functions REST API](https://cloudskills.io/blog/azure-functions-rest-api)

##### [Work with NoSQL data in Azure Cosmos DB](https://docs.microsoft.com/en-us/learn/paths/work-with-nosql-data-in-azure-cosmos-db/)