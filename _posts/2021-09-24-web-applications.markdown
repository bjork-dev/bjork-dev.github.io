---
author: Liam Björkman
tags: azure serverless containers FaaS
---



The following post describes a Todo application consisting of a front-end built on Razor Pages, an Azure Functions API as back-end and Azure CosmosDB as storage.

## The Applicaiton

The todo application has now been expanded to a ASP.NET Core Web App. The app lets us get todos from the database and add new ones. It can be accessed publicly on [https://bjorkdev.azurewebsites.net/](https://bjorkdev.azurewebsites.net/)



<img src="/img/todo1.png">

Clicking on details reveals more information about the object and lets us delete it as well.

<img src="/img/todo2.png">

At the bottom we can add a new todo to the list, where all fields are required.

<img src="/img/todo3.png">



## The Code

The following code blocks will describe the most interesting parts of the code.

### DataAccess Class

This is the data layer, where API request to the Azure Functions are sent. To avoid including the API keys in the code, I have created a `Azure KeyVault` and added the `"secrets"` there. Using C# it is very easy to retrieve these, thanks to the NuGet packages for `Azure.Keyvault` and `Azure.Identity`. The `DefaultAzureCredentialOptions { ExcludeVisualStudioCredential = true }` Sets our login to use our `az login` environment variable. When the application is deployed, this will be set to login as the application, which we need to grant access to in Azure. The `{ ExcludeVisualStudioCredential = true }` part is because of a known bug, where VS tries to sign in using the VS account instead, so we need to specify the exclusion of this.

<img src="/img/azureportal3.png">

```c#
public class DataAccess : IDataAccess
    {
        private readonly HttpClient _client = new();
        
        private readonly SecretClientOptions _options = new()
        {
            Retry =
            {
                Delay= TimeSpan.FromSeconds(2),
                MaxDelay = TimeSpan.FromSeconds(16),
                MaxRetries = 5,
                Mode = RetryMode.Exponential
            }
        };

        public async Task<Todo[]> GetAsync()
        {
            string code = await GetSecret("GetTodo");
            Todo[] todo = null;
            HttpResponseMessage response = await _client.GetAsync($"https://todo-project.azurewebsites.net/api/todo?code={code}");

            if (response.IsSuccessStatusCode)
                todo = await response.Content.ReadFromJsonAsync<Todo[]>();
            return todo;
        }

        public async Task<HttpResponseMessage> PostAsync(Todo todo)
        {
            string code = await GetSecret("AddTodo");
            var json = JsonSerializer.Serialize(todo);
            var data = new StringContent(json, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await _client.PostAsync($"https://todo-project.azurewebsites.net/api/todo?code={code}", data);
            return response;
        }

        public async Task<HttpResponseMessage> DeleteAsync(string id)
        {
            string code = await GetSecret("DeleteTodo");
            return await _client.DeleteAsync($"https://todo-project.azurewebsites.net/api/todo/{id}?code={code}");
        }

        private async Task<string> GetSecret(string secretName)
        {
            var azure = new DefaultAzureCredentialOptions { ExcludeVisualStudioCredential = true };
            var client = new SecretClient(new Uri("https://bjorkdev.vault.azure.net/"), new DefaultAzureCredential(azure), _options);
            KeyVaultSecret secret = await client.GetSecretAsync(secretName);
            return secret.Value;
        }
    }
```

### IndexModel class

Our Razor Pages only consists of `IndexModel`

```c#
 public class IndexModel : PageModel
    {
        private readonly ILogger<IndexModel> _logger;
        private readonly IDataAccess _client;

        public Todo[] Todo { get; set; }
        [BindProperty]
        public Todo AddTodo { get; set; }
        
        public IndexModel(ILogger<IndexModel> logger, IDataAccess client)
        {
            _logger = logger;
            _client = client;
        }

        public async Task<ActionResult> OnGetAsync()
        {
            Todo = await _client.GetAsync();
            return Page();
        }

        public async Task<RedirectToPageResult> OnPostAsync()
        {
            if (!ModelState.IsValid) return RedirectToPage("Index");
            await _client.PostAsync(AddTodo);
            return RedirectToPage("Index");
        }

        public async Task<RedirectToPageResult> OnPostDelete(string id)
        {
            var response = await _client.DeleteAsync(id);
            return RedirectToPage("Index");
        }
```

### HTML

And the actual HTML page (excluding the `_Layout`).

```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<!-- Todo Grid Items-->
<div class="row justify-content-center">

    @for (int i = 0; i < Model.Todo.Length; i++)
    {
        <!-- Todo Item -->
        <div class="col-md-6 col-lg-4 mb-5">
            <div class="portfolio-item mx-auto" data-bs-toggle="modal" data-bs-target="#bnag-@i">
                <div class="card" style="width: 18rem;">
                    <div class="card-body">
                        @Model.Todo[i].Title
                        <br/>
                        <strong>Click for details</strong>
                        <div class="portfolio-item-caption d-flex align-items-center justify-content-center h-100 w-100">
                            <div class="portfolio-item-caption-content text-center text-white"><i class="fas fa-plus fa-3x"></i></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    }
</div>

@for (int i = 0; i < Model.Todo.Length; i++)
{
    <div class="portfolio-modal modal fade" id="bnag-@i" tabindex="-1" aria-labelledby="@Model.Todo[i].Title" aria-hidden="true">
        <div class="modal-dialog modal-xl">
            <div class="modal-content">
                <div class="modal-header border-0"><button class="btn-close" type="button" data-bs-dismiss="modal" aria-label="Close"></button></div>
                <div class="modal-body text-center pb-5">
                    <div class="container">
                        <div class="row justify-content-center">
                            <div class="col-lg-8">
                                <!-- Portfolio Modal - Title-->
                                <h2 class="portfolio-modal-title text-secondary text-uppercase mb-0">@Model.Todo[i].Title</h2>
                                <!-- Icon Divider-->
                                <div class="divider-custom">
                                    <div class="divider-custom-line"></div>
                                    <div class="divider-custom-icon"><i class="fas fa-star"></i></div>
                                    <div class="divider-custom-line"></div>
                                </div>
                                <!-- Portfolio Modal - Text-->
                                <h4 class="mb-4">@Model.Todo[i].Description</h4>
                                <h3>Due by: @Model.Todo[i].DueBy</h3>
                                <form method="post">
                                    <button class="btn btn-danger" type="submit" asp-page-handler="Delete" asp-route-id="@Model.Todo[i].Id">Delete</button>
                                </form>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
}

<h2>Add A New Todo</h2>

<form method="post">
    Title:
    <input asp-for="AddTodo.Title" required class="form-control">
    <br />
    Description:
    <input asp-for="AddTodo.Description" required class="form-control">
    <br />
    Due By:
    <input asp-for="AddTodo.DueBy" required class="form-control">
    <br />
    <input type="submit" value="Add" class="btn btn-primary btn-lg btn-block" />
</form>
```

## Deployment

The deployment part will be divided in different levels (Bronze, Silver and Gold).

### Bronze

