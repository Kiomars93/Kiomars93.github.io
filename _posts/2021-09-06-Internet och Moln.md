---
layout: post
title:  "Kap 1 - 2021-09-06 Internet och Moln"
date:   2021-09-06 09:31:04 +0200
categories: jekyll update
image1:  /Image/aws1.PNG
image2:  /Image/aws2.PNG
image3:  /Image/azure.PNG
image4:  /Image/azure2.PNG
image5:  /Image/googlecloud.PNG
---

# Moln beskrivning:

Enkelt uttryckt är molntjänster leverans av datatjänster - inklusive servrar, lagring, databaser, nätverk, 
programvara, analys och intelligens - över Internet ("molnet") för att erbjuda snabbare innovation, 
flexibla resurser och stordriftsfördelar. Du betalar vanligtvis bara för molntjänster du använder, 
hjälper dig att sänka dina driftskostnader, 
driva din infrastruktur mer effektivt och skala när dina företags behov förändras.

Det finns tre distributionsmodeller:

Offentligt moln: Tjänsterna erbjuds via offentligt Internet och är tillgängliga för alla som vill köpa dem. 
Molnresurser som servrar och lagring ägs 
och drivs av en extern molntjänstleverantör och levereras via internet.

Privat moln: Ett privat moln består av beräkningsresurser som uteslutande används av användare från 
ett företag eller organisation. Ett privat moln kan 
finnas fysiskt i organisationens lokala datacenter eller hanteras av en tredje part.

Hybridmoln: Ett hybridmoln är en beräkningsmiljö som kombinerar ett offentligt moln 
och ett privat moln genom att tillåta att data och program delas mellan dem.

# Fördelar och nackdelar med moln:

Fördelar:
Vilka är fördelarna med molnbaserad databehandling?
Det finns flera fördelar med en molnmiljö jämfört med en fysisk miljö som Tailwind Traders kan ha nytta av när de migrerar till Azure.

Hög tillgänglighet: Beroende på vilket serviceavtal (SLA) du väljer, kan dina molnbaserade appar ge en kontinuerlig användarupplevelse utan märkbara avbrott, även om någonting går fel.

Skalbarhet: Appar i molnet kan skalas vertikalt och horisontellt:

Skala vertikalt när du behöver öka beräkningskapaciteten genom att lägga till mer RAM-minne eller fler processorer i en virtuell dator.
Skalning horisontellt ökar beräkningskapaciteten genom att lägga till instanser med resurser, till exempel att virtuella datorer läggs till i konfigurationen.
Elasticitet: Du kan konfigurera molnbaserade appar med automatisk skalning, så att apparna alltid har de resurser de behöver.

Flexibilitet: Distribuera och konfigurera molnbaserade resurser snabbt när kraven i apparna ändras.

Geo-distribution: Du kan distribuera appar och data till regionala datacenter över hela världen, så att kunderna alltid får bästa möjliga prestanda i sin region.

Haveriberedskap: När du använder molnbaserade tjänster för säkerhetskopiering, datareplikering och geo-distribution, kan du distribuera apparna och vara säker på att dina data är säkra vid ett eventuellt haveri.

Kapitalkostnader jämfört med operativa kostnader
Det finns två olika typer av utgifter som du bör tänka på:

CapEx (kapitalutgifter) är direkta utgifter för fysisk infrastruktur som sedan skrivs av över tid. 
CapEx-startkostnaden har ett värde som minskar med tiden.
OpEx (operativa kostnader) är utgifter för tjänster eller produkter som du debiteras för och betalar nu. 
Du kan dra av den här kostnaden under samma år den uppstår. Det finns ingen startkostnad 
eftersom du betalar för en tjänst eller produkt när du använder den.


Beskriva olika molntjänster:
Iaas, Paas, Saas och deras beskrivning och för och nackdelar


# Här nedan ser ni skillnadspriserna mellan dem stora moln-leverantör:

 Jag var dessvärre tvungen att dela upp den här bilden i två delar eftersom att jag ej kunde få med allting.

AWS:s EC2 tjänst ger massigt val av procesor,lagring, nätverk, operativsystem och inköpsmodell.

![Amazon webb server del 1]({{ page.image1 }})
![Amazon webb server del 2]({{ page.image2 }})

Beräknad pris på detta är 298.798 kr för tillfället. 
Detta är då om man har krav på 2st CPU och 8GB RAM och så la jag 30GB kapacitet på disken och man väljer Stockholm.

Jag valde 2 tjänster i hos azure och dessa är Virtual Machines (825,464 Kr) och Azure SQL Database (3553,88 Kr) med total summa av 4379,344. 
Dessa är för 1 instans av 2 CPU, 8GB RAM och 50GB disk. Valde 2 vCore instans för Azure SQL DB och därav fick jag det höga summan.
Valde Norway East för region.

![Azure del 1]({{ page.image3 }})
![Azure del 2]({{ page.image4 }})

Valde två tjänster för Google Cloud och dessa är Compute Engine (461.390 Kr) för själva hårdvarorna och Cloud SQL for MySQL (942.054 Kr) för DBhantering.
Valde Finland som region. Instanstypen e2-standard-2 innebär att vi får 2 CPU och 8 GB RAM. Så valde jag 10GB disk utrymme via DB:N.

![Google Cloud]({{ page.image5 }})