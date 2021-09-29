---
layout: post
title:  "Kap 4 - 2021-09-15 Serverless"
date:   2021-09-15 09:31:04 +0200
categories: jekyll update
image1:  /Image/AzureFunctions.png
image2:  /Image/Http trigger.png
image3: /Image/AzureResourcegroup&FunctionApp.png
image4: /Image/CalcAppFunction.png
image5: /Image/CalProject.png
image6: /Image/CalProjectURL.png
image7: /Image/CalProjectTestNRun.png
---


# Vad är Serverless och Function As A Service (FaaS)?


Serverless compute kan ses som en function som en service (FaaS) eller en microservice som finns på en molnplattform. Din affärslogik körs som funktioner och du behöver inte manuellt tillhandahålla eller skala infrastruktur. Molnleverantören hanterar infrastruktur. Din app skalas automatiskt ut eller ner beroende på belastning. Azure har flera sätt att bygga den här typen av arkitektur. De två vanligaste metoderna är Azure Logic Apps och Azure Functions.

*Skillnaden mellan FaaS och Serverless:*

Serverless och Functions-as-a-Service (FaaS) är ofta sammanblandade med varandra men sanningen är att FaaS faktiskt är en delmängd av serverless. Serverless är inriktat på alla tjänstekategorier, oavsett om det är beräkning, lagring, databas, meddelanden, api -gateways etc. där konfiguration, hantering och fakturering av servrar är osynliga för slutanvändaren. FaaS, å andra sidan, medans den kanske är den mest centrala teknologin inom serverless architectures, är fokuserad på det händelsedrivna computing paradigmet där applikationskod eller containers bara körs som svar på events eller requests.


Referens:
https://www.ibm.com/cloud/learn/faas#toc-faas-vs-se-V8QIDV-t

# Beskriv din/eran kalkylator.
- Koden?

Detta är koden jag har byggt i visual studio lokalt.
Det gjorde jag genom att välja ett Azure Functions projekt:

![Azure Functions Project]({{ page.image1 }})

Därefter valde jag Http trigger template:

![Http Trigger template]({{ page.image2 }})

Detta är en simpel kalkylator där jag har hård kodat tre stycken queries. Ett för namn och två stycken för talen så kommer att adderas!

Om man inte lägger någon query input så får man respektive meddelande:

*This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.*

Om du t.ex. lägger in ett namn med två olika värden så får du:

*Hello, Peter. Your total is 25.*

Resultatet ovan fick jag eftersom jag la in (&name=Peter&value1=10&value2=15) dessa.

``` csharp
public static class CalProject
    {
        [FunctionName("CalProject")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string name = req.Query["name"];
            string value1 = req.Query["value1"];
            string value2 = req.Query["value2"];

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            name = name ?? data?.name;

            var firstValue = Convert.ToDouble(value1);
            var secondValue = Convert.ToDouble(value2);

            var sum = firstValue + secondValue;

            string responseMessage = string.IsNullOrEmpty(name)
                ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
                : $"Hello, {name}. Your total is {sum}.";

            return new OkObjectResult(responseMessage);
        }
    }
```


- Hur har du/ni fått den att köra i Azure functions? Screenshots, scrips, pipelines

- Jag skapade ett github token och döpte det till AzureToken. 
- Skapade ett lokalt repo som döpte till CalculatorAzureApp. 
- Jag pushade upp dessa och gjorde min token secret och kopplat till mitt repo.
- La in följande kod i ordning:

``` cmd

# Enable authenticated git deployment in your subscription from a private repo.
az functionapp deployment source update-token --git-token (AzureTokens värde)

# Create a resource group.

az group create --name CalculatorGrp --location northeurope

# Create an Azure storage account in the resource group.

az storage account create --name kiokio007 --location northeurope 
--resource-group CalculatorGrp --sku Standard_LRS

# Create a function app with source files deployed from the specified GitHub repo.

az functionapp create 
--name CalcApp 
--storage-account kiokio007 
--consumption-plan-location northeurope 
--resource-group CalculatorGrp 
--deployment-source-url https://github.com/Kiomars93/CalculatorAzureApp.git 
--deployment-source-branch master 
--functions-version 2
```
- Installerade Azure Functions Tools v3.x windows 64 bitars (Detta är för att kunna köra func kommandon i terminalen)
- Gick in min azure portal konto genom att skriva (az login) i cmd kommandon.
- Gick in i Resource groups och valde min CalculatorGrp som är den gruppen jag skapade.

![Resource Group]({{ page.image3 }})

- Gick jag in på CalcApp function som är inuti min grupp.

![Functions App]({{ page.image4 }})

- Valde Function tabben

![Function Tab]({{ page.image5 }})

- Som ni ser så kan jag välja CalProject men dessa var tom från början. Jag matade in nedanstående kod för att skapa dessa:

``` cmd
# Skapa projektet
func init CalculatorAzureApp --dotnet
använde mkdir kommandon för därefter bytte jag direktory till CalculatorAzureApp
func new --name CalProject --template "HTTP trigger" --authlevel "function"
```

- Jag kan hämta URL till min kalkylator app och det fungerar exakt på samma sätt det gjorde
på min lokala app!

![FunctionApp URL]({{ page.image6 }})

Referens:
https://docs.microsoft.com/en-us/azure/azure-functions/scripts/functions-cli-create-function-app-github-continuous
https://docs.microsoft.com/sv-se/azure/azure-functions/functions-run-local?tabs=windows%2Ccsharp%2Cportal%2Cbash%2Ckeda#v2

# Hur har du testat applikationen?

- Jag trycker på (Code + Test) tabben som ni ser på ovanstående bild
- Trycker på Test/Run valet och som ni ser i exemplet nedan så har jag valt att göra en POST request
med querisen (name = Dev, value1 = 10, value2 = 2)

![CalProject Test/Run]({{ page.image7 }})

- Vilka säkerhets hot finns där till en applikation om din (beskriv minst en)? Och har du gjort något för att säkra dig emot dissa?

Attack Vectors:
Oanvända sidor ersätts med olänkade triggers, oskyddade filer och directories ändras till offentliga resurser,
som public buckets, Angriparna kommer att försöka identifiera felkonfigurerade funktioner med lång timeout eller
låg samtidighetsgräns för att orsaka en Denial of Service (DoS).
Dessutom funktioner som innehåller oskyddade secrets som nycklar och token i koden eller miljön kan så småningom resultera i
känsliga informationsläckage.

Lösning:
När jag gör min token secret så är det ett lösning för att inte bli attackerad.