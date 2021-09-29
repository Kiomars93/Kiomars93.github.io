---
layout: post
title:  "Kap 5 - 2021-09-20 Databaser i molnet"
date:   2021-09-20 09:31:04 +0200
categories: jekyll update
image1: /Image/CreateServer.png
image2: /Image/GetRequest.png
image3: /Image/PostRequest.png
image4: /Image/PostmanGet.png
image5: /Image/PostmanPost.png
image6: /Image/ClothesGroup.png
image7: /Image/DBItems.png
image8: /Image/AzureCloudAPI.png
image9: /Image/Triggers.png
---


# Beskriv kort applikationen, vad gör den?

Jag har gjort en klädapplikation där man ska kunna begära en GET och POST request för att kunna se det existerande klädprodukten och lägga till fler om
man vill.

# Beskriv koden & Databasen

- Skapade en Resource group direkt via azure portalen och döpte det till ClothesGroup.
- Skapade en Function App via azure poratalen och döpte det till ClothesApp
- Skapade en ny server via VS code:
    1. ![Create Server]({{ page.image1 }})
    2. Valde Core (SQL)
    3. Döpte accounten till clothes-account
    4. Valde serverless mode för capacity model
    5. På location så tog jag North Europe
- Högerklicka på servern jag precis skapat och valde Create Database
- Döpte mitt DB-namn till Clothes
- Döpte min container till my-container
- Döpte min partition key till /category
- Skapade mina lokala AzureApp project via VS code
- Publicera dett till min ClothesApp för att kunna köra det serverless


Just nu så ser min kod dessvärre väldigt rörigt ut. 

Koden nedan visar mitt get request där jag hämtar ner all data jag har i mitt Clothes DB:

![GetRequestAPI]({{ page.image2 }})

Här nedan fyller jag i det respektive queriesen som jag har valt och dessa är propertiesen i min Clothesmodel klass.

![PostRequestAPI]({{ page.image3 }})

Så ser det ut när vi hämtar ner all data från DB:n:

![POSTRequest]({{ page.image4 }})

ID:n visas null lokalt men i själva verket är det inga problem med ID:n eftersom att jag använder mig utav GUID och title postat också för varje
post-request jag gör. Här nedan är ett exempel på mitt POST-anrop:

![PostRequest]({{ page.image5 }})

Länken till både min GET och POST request via cloud är:

* https://clothesapp.azurewebsites.net/api/ClothesTriggerGet
* https://clothesapp.azurewebsites.net/api/ClothesTrigger?

Min resource group innehåller ClothesApp och inuti vår clothes-account så har jag min DB bl.a. andra funktioner:

![ResourceGroup]({{ page.image6 }})


Här ser ni innehållet i min Clothes-account:
I Data Explorer tabben så har jag Clothes DB:n och i items:en så ser man querisen och där inne ser man propsen
som vi har tidigare postat.

![DBContent]({{ page.image7}})

Referens: 
- https://docs.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-csharp?tabs=in-process&pivots=programming-runtime-functions-v3
- https://docs.microsoft.com/en-us/azure/azure-functions/functions-add-output-binding-cosmos-db-vs-code?tabs=in-process&pivots=programming-language-csharp


# Hur har du/ni fått den att köra i Azure functions? Screenshots, scripts

Som jag nämnde förr så har jag publishat min lokala fil till min ClothApp som är min azure functions App
i azure portalen. 

Här nedan ser ni båda API:er som jag har publishat till min ClothesApp.

![AzureCloudAPI]({{ page.image8 }})

Här ser ni triggers:en som jag har publishat i min azure portal. Detta ser man i ClothesApp => Function tab.

![CloudTriggers]({{ page.image9}})


# Hur har du tänkt runt uppdatering av databasen ifall scheman ändras? Migrations?

Detta sker via Change Feed som lyssnar efter ändringar i Cosmos DB -samlingen. Varje gång ett nytt dokument läggs till i samlingen (dvs. en händelse inträffar som en användare som tittar på ett objekt, lägger till ett objekt i sin kundvagn eller köper ett objekt), kommer change feed att utlösa en Azure -funktion. Det är så
cosmos DB hantering migrering på sitt sätt.

Referens: https://docs.microsoft.com/en-us/samples/azure-samples/azure-cosmos-db-change-feed-dotnet-retail-sample/azure-cosmos-db-change-feed-dotnet-retail-sample/



# Vad skulle det kosta att driva detta? Tänk gärna två scenarier: Nästan ingen använadere och jätte jätte mycket användare 

1. För en användare:

    * Azure Cosmos DB - Om vi väljer serverless för Database operations, Single Region Write, 1 milion RUs, North Europe regionen kommer det att kosta 2,4 kr/månad
    * Azure Functions - , North Europe, 128 GB memory size, , Consumption tier för Azure function så kostar det 0 kr/månad.
     Total summa blir: 2,4 kr/månad.

2. För flera användare:

    * Azure Cosmos DB - Om vi väljer Autoscale provisioned throughput för Database operations, Multiple Region Write, 4000 RU/S (Max Request per second), North Europe regionen kommer det att kosta 4050 kr/månad
    * Azure Functions - Premium tier, North Europe, EP3: 4 Cores(s), 14 GB RAM, 250 GB Storage för Azure function så kostar det 10500 kr/månad. Total summa blir: 14600 kr/månad.

