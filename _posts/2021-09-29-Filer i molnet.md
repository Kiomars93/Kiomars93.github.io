---
layout: post
title:  "Kap 8 - 2021-09-29 Filer i molnet"
date:   2021-09-29 09:31:04 +0200
categories: jekyll update
image1: /Image/Molnbilder/blobflow.png
image2: /Image/Molnbilder/blobconnectionstring.png
image3: /Image/Molnbilder/blobcontainer.png
image4: /Image/Molnbilder/encryptionatrest.png
image5: /Image/Molnbilder/encryptioninflight.png
image6: /Image/Molnbilder/keyvault.png
image7: /Image/Molnbilder/Homepage.png
image8: /Image/Molnbilder/Picturestab.png
---

# Beskriv kort applikationen, vad gör den?

Jag har skapat en consoleapp och döpt den till Azureblop. Det appen gör är att det skickar upp min bilden till Azure Storage och Kan också
Dra ner bilden från molnet.

# Gör en diagram (använd eg draw.io) som visar hur data flyter.

Här nedan ser ni dataflowet som skrev från Azure Storage account till filen:

![Dataflow]({{ page.image1 }})

# Beskriv koden

- Skapade en ny Storage account och det gjorde jag genom att välja alternativ knappen längst upp till vänster => Create a resource => Storage account => Create.
- Valde mitt student konto, valde ContosoResourceGroup som Resource group, namn på mitt storage account blobby007, North Europe, Standard performande, för Redundancy så valde jag Locally-redundant storage (LRS).
- Gick in på min storage account och => Access Keys och kopierade min connectionstring under mitt Key1 som ni ser på bilden nedan:

    ![Connectionstring]({{ page.image2 }})

- Skapade en blob genom att gå in på min storage acount => Containers taben under Data storage => Valde Container och därefter skrev jag in mitt containersnamn. Ni ser nu i detta fallet att jag har skapat en container med exakt det namnet och för visibiliteten så valde jag blob:

    ![Container]({{ page.image3 }})

- Här nedan ser ni koden jag har skrivit för min console app demo och som ni ser så har lagt till mitt containernamn. Kopplade min conenctionstring genom att skriva in detta commandot i pmc och la till Azure.Storage.Blobs nuget packet för att ha tillgång till Blobs funktionerna i koden:

``` pmc

setx AZURE_STORAGE_CONNECTION_STRING "<yourconnectionstring>"

dotnet add package Azure.Storage.Blobs

```
- Här nedan ser ni att jag har valt att min bild som heter Network1.jpg i respektive filepath ska skickas upp till min azure storage account.
- Detta görs via CreateBlob metoden och som det heter så skapar det en bilden i min container och därefter får jag ett meddelande att Blob är skapat.
- Jag har lagd till en GetBlob metodn som drar ner de existerande bilderna i azure storage account.

``` csharp
class Program
    {
        static string connectionstring = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
        static string containerName = "justacontainer";
        static string filename = "Network1.jpg";
        static string filepath = "./data/Network1.png";
        static void Main()
        {
            CreateBlob().Wait();
            GetBlob().Wait();
        }
        static async Task CreateBlob()
        {
            BlobServiceClient blobServiceClient = new BlobServiceClient(connectionstring);
            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);
            BlobClient blobClient = containerClient.GetBlobClient(filename);
            using FileStream uploadFileStream = File.OpenRead(filepath);
            // Raden nedan möjliggör så att man kan displaya bilden direkt via browsers istället för
            // det ska laddas ner
            var blobHttpHeader = new BlobHttpHeaders { ContentType = "image/jpg" };
            await blobClient.UploadAsync(uploadFileStream, new BlobUploadOptions { HttpHeaders = blobHttpHeader });
            //await blobClient.UploadAsync(uploadFileStream, true);
            uploadFileStream.Close();
            Console.WriteLine("Blob created!");
        }

        static string downloadpath = "./data/Network1.png";
        static async Task GetBlob()
        {
            BlobServiceClient blobServiceClient = new BlobServiceClient(connectionstring);
            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);
            BlobClient blob = containerClient.GetBlobClient(filename);
            BlobDownloadInfo blobdata = await blob.DownloadAsync();
            using (FileStream downloadFileStream = File.OpenWrite(downloadpath))
            {
                await blobdata.Content.CopyToAsync(downloadFileStream);
                downloadFileStream.Close();
            }
            Console.WriteLine("Blob Downloaded!");
        }
    }

```

- Om jag inte vill dra ner filen via min GetBlob metoden så kan jag göra det via mitt ASP MVC projekt.
- Har döpt det till BlobMVC. Jag kan böra med att visa hur jag kopplat min Connectionstring till projektet:
- Detta är innehållet i min appsettings.json fil:

``` json
{
  "AZURE_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=https;AccountName=bloby007;AccountKey=CtPLsOhx8OC2ZJxMQbkZ5kwjzotsjd7uuDeAgcetq2/2U4Dz4g5sQgFVHuCTDzXVxJN61fmqVi/b8l4ZuD2L9Q==;EndpointSuffix=core.windows.net",
}
```
- Innehållet som jag har injectat via min startup.cs:

``` csharp
public void ConfigureServices(IServiceCollection services)
        {
            // DI för min connectionstring
            services.AddSingleton(x => new BlobServiceClient(Configuration.GetValue<string>("AZURE_STORAGE_CONNECTION_STRING")));
            // DI för min BlobService och IBlobService
            services.AddSingleton<IBlobService, BlobService>();
        }

```

- Därefter har vi routingen:
    Jag har skapat en map i min View map och har döpt det till BlobExplorer och skapat en Index.cshtml fil där jag har döpt 
    till Pictures som jag kommer att visa lite längre ner:

``` csharp
<li>
     <a class="nav-link text-dark" asp-area="" asp-controller="BlobExplorer" asp-action="Index">Pictures</a>
</li>

```
- Innehåller på min controller som jag har skapat:
    Jag använder mig av min IBlovService interface och anropar ListBlobAsync metoden som hämtar ner alla mina bilder via azure blobs.

``` csharp
[Route ("[Controller]")]
    public class BlobExplorerController : Controller
    {
        private readonly IBlobService _blobService;
        public BlobExplorerController(IBlobService blobService)
        {
            _blobService = blobService;
        }



        //[HttpGet("list")]
        public async Task<IActionResult> Index()
        {
            return View(await _blobService.ListBlobAsync());
        }
    }

```
- Innehållet inuti min Index.cshtml fil där jag visar mina bilder:

``` csharp
@model List<string>
@{
    ViewData["Title"] = "Pictures";
}

<h1>Picture list from my Azure Storage account container!</h1>

@foreach (var blobItem in Model)
{
    <img src="@("https://bloby007.blob.core.windows.net/justacontainer/" + blobItem)" alt=image />
}

```

- Logiken bakom min ListBlobAsync metod:

``` csharp
// Själva interfacet:

public interface IBlobService
    {
        public Task<IEnumerable<string>> ListBlobAsync();
    }

// Klassen som implementerar interfacet:
// Som ni ser så väljer det min blob container och hämtar alla filer inuti lägger till 
// min blobsList lista.
public class BlobService : IBlobService
    {
        private readonly BlobServiceClient _blobServiceClient;

        public BlobService(BlobServiceClient blobServiceClient)
        {
            _blobServiceClient = blobServiceClient;
        }

        public async Task<IEnumerable<string>> ListBlobAsync()
        {
            var containerClient = _blobServiceClient.GetBlobContainerClient("justacontainer");
            var blobsList = new List<string>();
            await foreach (var blobItem in containerClient.GetBlobsAsync())
            {
                blobsList.Add(blobItem.Name);
            }
            
            return blobsList;
        }
    }

```

- Så här ser det min web app ut:

![Homepage]({{ page.image7 }})

- Trycker man på picturestaben så ser man bilderna (I mitt fall är det enbart en bild):

![Picturestab]({{ page.image8 }})

# Vad skulle det kosta att driva en applikation som spara och läser filer i Azure? Låt oss säga att man ska bygga en applikation som shutterstock. Vad skulle hända med kostnad över tid om du har 1000 använder som lägger upp 100 MB nya bilder varje dag (med din konsoll app), och alla bilder som finns i din blob laddas ner tre gångar per dag (med ditt web UI).

Om du maximerar en Storage accounts på Block Blob Storage type, Premium perforamance, North Europe med, Zone-redundant storage (ZRS) och 1000 GB kapacitet så
hamnar du på ca 1050Kr/Månaden.



# Explain with your own words what Microsoft does to secure your blob data:

Det finns tre viktiga områden och dessa är:

1. Hur Azure krypterar data i vila:
     Kryptering av data i vila skyddar lagrad information från oönskad åtkomst. Till exempel kan viloläge-kryptering skydda innehållet på din hårddisk om den försvann eller stals.

    ![Encryption at Rest ]({{ page.image4 }})

2. Hur Azure krypterar data under flygning:
     Azure-kryptering av data In-flightData under flygning (till exempel dataöverföringar över det öppna internet till cloud storage, webbsessioner och andra typer av datarörelser) måste skyddas mot avlyssning, interception och tampering. Alla sessioner med Azure -tjänster och datacenter är säkrade med hjälp av Transport Layer Security (TLS) kryptografiskt protokoll och Forward Secrecy (även känt som Perfect Forward Secrecy eller PFS), ett nyckelavtalsprotokoll:
    
    ![Encryption In-flight ]({{ page.image5 }})


3. Azure Key mangement:
    Azure Key Vault som rekommenderad är en central källa för nyckellagring, generation, loggning och hantering. Microsoft beskriver Key Vault -erbjudandet: “Azure Key Vault hjälper till att skydda kryptografiska nycklar och hemligheter som används av molnprogram och tjänster. Genom att använda Key Vault kan du kryptera nycklar och hemligheter (t.ex. autentiseringsnycklar, lagringskontonycklar, datakrypteringsnycklar, .PFX -filer och lösenord) med hjälp av nycklar som skyddas av hårdvarusäkerhetsmoduler (HSM). För ökad säkerhet kan du importera eller generera nycklar i HSM. Om du väljer att göra detta, bearbetar Microsoft dina nycklar i FIPS 140-2 nivå 2-validerade HSM: er (hårdvara och fast programvara). ”Grafiken nedan illustrerar fördelarna och användningsfallen för Azure Key Vault:

    ![Azure Key Vault]({{ page.image6 }})


Referens:
- https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet
- https://www.c-sharpcorner.com/article/reading-and-writing-into-azure-blob-storage-using-net-core-console-app-and-c-sharp/
- https://cloudacademy.com/blog/how-does-azure-encrypt-data/
- https://www.youtube.com/watch?v=9ZpMpf9dNDA&t=350s&ab_channel=NickChapsas