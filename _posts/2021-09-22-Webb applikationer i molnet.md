---
layout: post
title:  "Kap 6 - 2021-09-22 Webb applikationer i molnet"
date:   2021-09-08 09:31:04 +0200
categories: jekyll update
image1: /Image/AzureWebApp/ClothesOverview.png
image2: /Image/AzureWebApp/AddClothes.png
image3: /Image/AzureWebApp/ClothesDBQueries.png
image4: /Image/AzureWebApp/CosmoRazor.png
---

# Beskriv kort applikationen, vad gör den?

Jag har lagt till ett nytt ASP.NET Core Web Application Razor Page template projekt till min kläd app där jag kan se min klädlista genom att anropa en GET-method för att hämta alla queries i min Clothes DB samt kunna Tillägga Queries till min DB genom att skapa en POST-anrop via min POST-method.

# Beskriv koden

Det min Razor projekt gör att det helt enkelt kan kommunicera med DB:N som är i mitt azure-account. Det första jag gjorde var att:
- Configurera appen genom att tillägga mitt account-URI, PRIMARY-Key, DB namn och containername och döpt dessa till CosmosDb:
``` json
"CosmosDb": {
    "Account": "<Endpoint URI of your Azure Cosmos account>",
    "Key": "<PRIMARY KEY of your Azure Cosmos account>",
    "DatabaseName": "Clothes",
    "ContainerName": "my-container"
  }
```
- Därefter har jag lagt till en dependency injection i ConfiguresServices metoden i startup.cs filen under AddRazorPages() DI:N:
  Ni ser ju att jag har anvnänt min appsettings.json CosmodDb namn i min DI.

``` csharp
public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSingleton<ICosmosDbService>(InitializeCosmosClientInstanceAsync(Configuration.GetSection("CosmosDb")).GetAwaiter().GetResult());
        }
```

- Sedan har jag också lagd till en ytterligare metod och vad det gör är att det läser configuration och initialiserar klienten:

```csharp
private static async Task<CosmosDbService> InitializeCosmosClientInstanceAsync(IConfigurationSection configurationSection)
        {
            string databaseName = configurationSection.GetSection("DatabaseName").Value;
            string containerName = configurationSection.GetSection("ContainerName").Value;
            string account = configurationSection.GetSection("Account").Value;
            string key = configurationSection.GetSection("Key").Value;
            Microsoft.Azure.Cosmos.CosmosClient client = new Microsoft.Azure.Cosmos.CosmosClient(account, key);
            CosmosDbService cosmosDbService = new CosmosDbService(client, databaseName, containerName);
            Microsoft.Azure.Cosmos.DatabaseResponse database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
            await database.Database.CreateContainerIfNotExistsAsync(containerName, "/category");

            return cosmosDbService;
        }
```

- Installera Nuget Paketet Microsoft.Azure.Cosmos
- Skapade två ModelPage och döpt dessa till Create som hanterar mina POST ärenden och ClothesOverview som hanter GET
- Skapat en CosmosDbService klass och en ICosmosDbService interface
- I CosmosDbService så har jag DI:n, contructor som tar int dbCliten, dbnam och containernamnet, GetItemsAsync metod för att ta emot alla queries och AddItemsAsync för att tillägga en query. Koden jag beskrev kan ni se nedan:

``` csharp
public class CosmosDbService : ICosmosDbService
    {
        private Container _container;

        public CosmosDbService(CosmosClient dbClient, string databaseName, string containerName)
        {
            this._container = dbClient.GetContainer(databaseName, containerName);
        }


        public async Task<IEnumerable<ClothesWebModel>> GetItemsAsync(string queryString)
        {
            var query = this._container.GetItemQueryIterator<ClothesWebModel>(new QueryDefinition(queryString));
            List<ClothesWebModel> results = new List<ClothesWebModel>();
            while (query.HasMoreResults)
            {
                var response = await query.ReadNextAsync();

                results.AddRange(response.ToList());
            }

            return results;
        }

        public async Task AddItemAsync(ClothesWebModel clothesModel)
        {
            await this._container.CreateItemAsync<ClothesWebModel>(clothesModel, new PartitionKey(clothesModel.Category));
        }
    }

// Och Interfacet
public interface ICosmosDbService
    {
        Task<IEnumerable<ClothesWebModel>> GetItemsAsync(string query);
        Task AddItemAsync(ClothesWebModel clothesModel);
    }

```

- Min ClotheOverview och Create PageModel :

``` csharp
public class CreateModel : PageModel
    {
        private readonly ICosmosDbService _cosmosDbService;
        public CreateModel(ICosmosDbService cosmosDbService)
        {
            _cosmosDbService = cosmosDbService;
        }

        [BindProperty]
        public ClothesWebModel Clothes { get; set; }

        public async Task<IActionResult> OnPostAsync()
        {

            Clothes.Id = Guid.NewGuid().ToString();
            await _cosmosDbService.AddItemAsync(Clothes);

            return Redirect("/Index");
        }
    }
```

``` csharp
@page
@model CreateModel
@{
    ViewBag.Title = "Add more Clothes";
}

<h2>Add Clothes</h2>

<div class="w-50 justify-content-md-center" style="margin-left:25%">
    <h2 class="display-4" style="text-align: center">Add a new Clothes item</h2>

    <form method="post" class="align-items-center" style="align-content: center">
        @Html.LabelFor(a => a.Clothes.Title)
        <input asp-for="Clothes.Title" required class="form-control">
        <br />
        @Html.LabelFor(a => a.Clothes.Description)
        <input asp-for="Clothes.Description" required class="form-control">
        <br />
        @Html.LabelFor(a => a.Clothes.Color)
        <input asp-for="Clothes.Color" required class="form-control">
        <br />
        @Html.LabelFor(a => a.Clothes.Price)
        <input asp-for="Clothes.Price" required class="form-control">
        <br />
        <button type="submit" class="btn btn-dark w-25">Submit</button>
    </form>
</div>

<div>
    @Html.ActionLink("Back to List", "Index")
</div>
<script src="~/bundles/jqueryval"></script>
```

``` csharp
public class ClothesOverviewModel : PageModel
    {
        private readonly ICosmosDbService _cosmosDbService;
        public ClothesOverviewModel(ICosmosDbService cosmosDbService)
        {
            _cosmosDbService = cosmosDbService;
        }

        [BindProperty]
        public List<ClothesWebModel> ClothesList { get; set; } = new List<ClothesWebModel>();
        [BindProperty]
        public ClothesWebModel Clothes { get; set; }

        public async Task<IActionResult> OnGet()
        {
            var result = await _cosmosDbService.GetItemsAsync("SELECT * FROM c");

            foreach (var clothItem in result)
            {
                ClothesList.Add(clothItem);
            }

            return Page();
        }
    }
```

``` csharp
@page
@model ClothesOverviewModel
@{
    ViewBag.Title = "Clothes Overview";
}

<h2>Clothes Overview</h2>

<table class="table">
    <tr>
        <th>
            @Html.DisplayNameFor(model => model.Clothes.Title)
        </th>
        <th>
            @Html.DisplayNameFor(model => model.Clothes.Description)
        </th>
        <th>
            @Html.DisplayNameFor(model => model.Clothes.Color)
        </th>
        <th>
            @Html.DisplayNameFor(model => model.Clothes.Price)
        </th>
        <th></th>
    </tr>


    @foreach (var item in Model.ClothesList)
    {
<tr>
    <td>
        @Html.DisplayFor(modelItem => item.Title)
    </td>
    <td>
        @Html.DisplayFor(modelItem => item.Description)
    </td>
    <td>
        @Html.DisplayFor(modelItem => item.Color)
    </td>
    <td>
        @Html.DisplayFor(modelItem => item.Price)
    </td>
```

- Här ser ni min query lista när man trycker på Overview Tabben på sidans headers:

    ![ClothesOverview]({{ page.image1 }})

- Fyllde i propertiesen via min Add More Items tab (Har lagt till en required attribute för respektive properties vilket innebär att allt måsta vara ifylld annars så skapas det ej någon query till min DB):

    ![AddClothes]({{ page.image2 }})

- Här ser innehållet på det som jag har postat:

    ![ClothesDBQueries]({{ page.image3 }})



# Hur har du/ni fått den att köra i Azure App Service? Screenshots, scrips, pipelines


- Efter att jag gjorde klart koden lokalt så laddade jag ner Azure Service extention. 
- Använde mitt azure konto och skapade ett nytt azure Web App via kod och gjorde följande:
    1. Skapade en ny web app och döpte det till CosmoRazor
    2. Valade .Net Core 3.1 (LTS)
    3. Linux
    4. F1 Free pricing tier
    5. North Europe

- Då fick jag fram:

![CosmoRazor]({{ page.image4 }})
Min domänlänk är: https://cosmorazor.azurewebsites.net/

- Jag skickade upp all kod via azure service extension.
- Jag tryckte på Azure ikonen
- Valde min CosmoRazor i min App Service extension
- Högerklicka på CosmoRazor och valde Deploy to Web App
- Valde Deploy därefter och därefter så var allting klart!

# Vad skulle det kosta att driva detta? Tänk gärna två scenarier: Nästan ingen använadere och jätte jätte mycket användere

1. För en användare:

    * Azure Cosmos DB - Om vi väljer serverless för Database operations, Single Region Write, 1 milion RUs, North Europe regionen kommer det att kosta 2,4 kr/månad
    * Azure Functions - , North Europe, 128 GB memory size, , Consumption tier för Azure function så kostar det 0 kr/månad.
    * App Service - North Europe, F1 FREE instance, Windows kostar ingenting.  Total summa blir: 2,4 kr/månad.

2. För flera användare:

    * Azure Cosmos DB - Om vi väljer Autoscale provisioned throughput för Database operations, Multiple Region Write, 4000 RU/S (Max Request per second), North Europe regionen kommer det att kosta 4050 kr/månad
    * Azure Functions - Premium tier, North Europe, EP3: 4 Cores(s), 14 GB RAM, 250 GB Storage för Azure function så kostar det 10500 kr/månad. 
    * App Service - North Europe, Premium V3, Windows, 5 instance av P3V3: 8 Cores(s), 32GB RAM, 250 GB Storage kostar 41400/månad Total summa blir: 56000 kr/månad.



Referens:
- https://docs.microsoft.com/en-us/azure/app-service/quickstart-dotnetcore?tabs=netcore31&pivots=development-environment-vscode
- https://docs.microsoft.com/en-us/azure/cosmos-db/sql/sql-api-dotnet-application