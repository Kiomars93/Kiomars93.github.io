---
layout: post
title:  "Kap 10 - 2021-10-06 Skalning, upp och ut"
date:   2021-10-06 09:31:04 +0200
categories: jekyll update
image1: /Image/Scalability/ScaleOutAppService.png
image2: /Image/Scalability/ScaleUpWithVM.png
---


# Förklara skildnad på att skala en applikation horisontellt och vertikalt

1. Horisontellt: 

- Fördelar med horisontell skalning
    Skalning är enklare ur ett maskinvaruperspektiv - Allt horisontellt skalningskrav som du behöver göra är att lägga till ytterligare maskiner i din nuvarande pool. Det eliminerar behovet av att analysera vilka systemspecifikationer du behöver uppgradera.
    Färre perioder av stillestånd - Eftersom du lägger till en maskin behöver du inte stänga av den gamla maskinen medan du skalar. Om det görs effektivt kan det aldrig behövas driftstopp och kunderna är mindre benägna att påverkas.
    Ökad motståndskraft och feltolerans - Att förlita sig på en enda node för alla dina data och operationer riskerar att förlora allt när det misslyckas. Att distribuera det mellan flera noder sparar dig från att förlora allt.
    Ökad prestanda - Om du använder horisontell skalning för att hantera din nätverkstrafik möjliggör det fler slutpunkter för anslutningar, med tanke på att belastningen kommer att delegeras mellan flera maskiner.

- Nackdelar med horisontell skalning
    Ökad komplexitet av underhåll och drift - Flera servrar är svårare att underhålla än en enda server är. Dessutom måste du lägga till programvara för lastbalansering och eventuellt virtualisering. Säkerhetskopiering av dina maskiner kan också bli lite mer komplex. Du måste se till att noder synkroniserar och kommunicerar effektivt.
    Ökade initiala kostnader - Att lägga till nya servrar är mycket dyrare än att uppgradera gamla.

2. Vertikalt:

- Fördelar med vertikal skalning
    Kostnadseffektivt-Uppgradering av en befintlig server kostar mindre än att köpa en ny. Dessutom är det mindre sannolikt att du lägger till ny programvara för säkerhetskopiering och virtualisering när du skalar vertikalt. Underhållskostnaderna kan också förbli desamma.
    Mindre komplex processkommunikation - När en enda nod hanterar alla lager i dina tjänster behöver den inte synkronisera och kommunicera med andra maskiner för att fungera. Detta kan resultera i snabbare svar.
    Mindre komplicerat underhåll - Underhållet är inte bara billigare utan det är också mindre komplext på grund av antalet noder du behöver hantera.
    Mindre behov av programvaruändringar - Det är mindre troligt att du ändrar hur programvaran på en server fungerar eller hur den implementeras.

- Nackdelar med vertikal skalning
    Högre möjlighet för stillestånd - Om du inte har en reservserver som kan hantera operationer och förfrågningar, behöver du betydligt stillestånd för att uppgradera din maskin.
    Enstaka felpunkt - Att ha alla dina funktioner på en enda server ökar risken att förlora all din data om ett maskin- eller programvarufel skulle inträffa.
    Uppgraderingsbegränsningar - Det finns en begränsning för hur mycket du kan uppgradera en maskin. Varje maskin har sin tröskel för RAM, lagring och processorkraft.


- Vilket ska du välja och när?

    Både horisontell och vertikal skalning har sina egna fördelar och begränsningar. Eftersom det inte finns en lösning som passar alla för organisationer måste du skala efter dina behov och resurser. Här är några faktorer att tänka på tillsammans med vilken typ av skalning som passar situationen bäst:

    Kostnad - Initial hårdvarukostnad för horisontella uppgraderingar är högre. Om du arbetar med en stram budget och behöver lägga till fler resurser till din infrastruktur snabbt och billigt, kan vertikal skalning vara det bästa alternativet för dig.
    Framtidssäkring - Om du lägger till ytterligare uppdaterade maskiner genom horisontell skalning ökar organisationens övergripande prestandatröskel. Det finns en gräns för hur mycket du kan skala en enda nod vertikalt och den kanske inte kan hantera framtidens krav.
    Topografisk distribution - Om du planerar att ha rikstäckande eller globala kunder är det orimligt att förvänta sig att alla får tillgång till dina tjänster från en enda maskin på en enda plats. I en sådan situation måste du skala dina resurser horisontellt för att behålla ditt servicenivåavtal (SLA).
    Pålitlighet - Horisontell skalning kan erbjuda dig ett mer tillförlitligt system. Det ökar redundansen och ser till att du inte litar på en enda maskin. Om en maskin misslyckas kan en annan kanske ta upp slacken tillfälligt.
    Uppgraderbarhet och flexibilitet - Om du kör applikationens nivåer på enskilda maskiner är de lättare att koppla från och uppgradera utan driftstopp.
    Prestanda och komplexitet - Prestanda beror på hur dina tjänster fungerar och hur de är sammankopplade. Enkla okomplicerade applikationer kommer inte att gynnas mycket av att köras på flera maskiner. Det kan faktiskt försämra dess kvalitet. Ibland är det bättre att lämna programmet som det är och uppgradera hårdvaran för att möta efterfrågan. Horisontell skalning kan kräva att du skriver om koden eller lägger till en virtuell dator som förenar alla servrar.

# Hur påverker det kostnad att ha en applikation Azure som skalar horisontalt vs vertikalt?

Azure App Service har begränsningar jämfört med virtuella Azure VMs när det gäller skalbarhet. 
Därför är Azure VMs föredragna för appar som har utrymme att expandera för framtiden.

- För en app service:

Behöver man flera instanser så en nog app service bättre att välja.
Om man tar en standard tier så har man på max 30 instanser som ni ser på bilden och då hamnar man på ca
640 kr/månaden.

![App Service Chart]({{ page.image1 }})

- En en virtuell maskin:

Medans vill man något kraftfull så är nog VM bättre.
Jag valde i detta fal en 8v CPUs, 32 GB RAM, 200 GB Temporary Storage.
Då kan man hamna på ca 5000 kr /månaden.  

![Virtual Machine Price List]({{ page.image2 }})


# Inte alla Azure Service App plans ger möjlighet att skala, vilka ger vilka möjligheter?

Om ni kollar igen på bilden ovan så ser vi att det är enbart Standart, Premium och Isolated tier som supportar skalbarhet.

Referens:
- https://www.cloudzero.com/blog/horizontal-vs-vertical-scaling
- https://karansinghreen.medium.com/azure-virtual-machine-or-azure-app-service-which-one-should-you-choose-d4ba7d4a120d
- https://azure.microsoft.com/en-us/pricing/details/app-service/windows/
- https://azure.microsoft.com/en-us/pricing/details/virtual-machines/windows/
