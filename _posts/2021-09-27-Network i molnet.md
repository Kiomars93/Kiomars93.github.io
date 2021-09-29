---
layout: post
title:  "Kap 7 - 2021-09-27 Nätverk i molnet"
date:   2021-09-27 09:31:04 +0200
categories: jekyll update
image1: /Image/Network/Network1.png
image2: /Image/Network/Network2.png
image3: /Image/Network/Network3.png
image4: /Image/Network/Network4.png
image5: /Image/Network/VPC.png
---

# Varför är det bra att använda sig av Azure service bus:

*Kortbeskrivning:*

Om vi skickar något från punkt 1 till punkt 2 den mellan grejen som är själva kommunikationen en är asynkron process. 
Det message som vi publishar från punkt 1 är väldigt durable, vilket innebär när dem hamnar i den mellan grejen som förloras aldrig messagen. Däremot går dem till DL (DeadletterQueue) eller så konsumeras dem av punkt 2 (applikationen)


![Sammanfattning]({{ page.image1 }})

*Lite kort förklaring med olika workflows och därefter komma jag gå igenom varför det är bra att använda sig utav Azure Service Bus:*


- Workflow Scenario 1: Låt oss säga att vi ska boka en flygresa via en app. Det här appen innehåller web, ordering service internt, payment och inget av dessa är asynkront. Appen kommer att köras långsammare. T.ex. när den hantera transaktioner så kommer det att ta längre för att processen ska bli utförd.

![Workflow1]({{ page.image2 }})

- Workflow Scenario 2: Det som skulle vara ett bättre alternativ är att användaren bokar sitt flyg via ett web interface och därefter vill göra många anrop, som t.ex. payment, orders och boking API. Det som är ett problem är då att vi återigen måste vänta på svar från vår API och enda skillnaden är att vi flyttade order API externt.

![Workflow2]({{ page.image3 }})

- Workflow Scenario 3: Det här vår enterprise grad Azure service bus kommer till en användning. Web api anropar booking api och det anropar payment api för credential och etc så fort booking får in ett svar från payment så booking service vill publish:a en message i azure service bus och därefter publiceras det till antigen queue eller en topic. Eventuellt t.ex. en order process service.exe fil konsumerar azure service bus som plockar upp message och anropa alla nödvändiga anrop för att t.ex. seats api, om nån har bokat nåt billigare etc. från Booking api publishar messages till exe.fil till alla andra anrop sker asynkront. När allting är klart så får vi ett email med alla detaljer. Det har många fördelar som när om något failas i orderprocessfilen blir det retried från queue eftersom att det går tbx till queue för republish. Om allting failas så hamnar det i DL.    

![Workflow3]({{ page.image4 }})

Några av Azure Service Bus features:

- Queue:
    FIFO (First in, first out) approach:
    Punkt 1 till queue till punkt 2/ Consumer där consumer är en typ av app. Det betyder i vårt exempel du kan ha flera instanser av vår orderprocessing fil men du kan ej ha andra tjänster förutom vår existerade tjänst för att lyssna på samma queue. Då kommer du att förlora message därav kan du ha enbart en typ av app som sagt.


- Topics /Subs:
    User publisher till topics och topics kan har subs. Message som skickas till topics skickas till varsitt subs (vilket kan vara vilken service som helst) som lyssnar på det. Det som skickas till topics skickar till alla subs och vill du t.ex. begränsa vissa event i någon särkilds fil så kan använda dig utav filters.

- Dead letter queue feature in azure service bus:
    Om du publishar något till queue och något går ej som det ska så skickas det tillbaka till en separat queue som vi kallar för dead letter queue. DL stannar där hur länge du vill att det ska vara. Det ger dig möjligheten att skicka om eller kolla om någon slags bugg innan du skickar om.


# Azure private link:

En privat endpoint är ett network interface som ansluter dig privat och säkert till en service som drivs av Azure Private Link. Den privata endpoint:en använder en privat IP -adress från ditt VNet, vilket effektivt tar tjänsten till ditt VNet. All trafik till tjänsten kan routas genom den privata endpoint:en, så inga gateways, NAT -enheter, ExpressRoute eller VPN -anslutningar eller offentliga IP -adresser behövs. Trafik mellan ditt virtuella nätverk och tjänsten går över Microsofts backbone network, vilket eliminerar exponering från det public Internet. Du kan ansluta till en instans av en Azure resource, vilket ger dig den högsta nivån av granularitet i access control.

Med Azure Private Link Service kan du ha access till Azure -services (till exempel Azure Service Bus, Azure Storage och Azure Cosmos DB) och Azure hosted customer/partner services för en privat endpoint i ditt virtuella nätverk. Vilket är perfekt i vårt fall om vi ska ha våra messages hidden.



# Virtual Private Cloud:

Ett virtuellt privat moln (VPC) är ett säkert, isolerat private cloud som finns i ett public cloud. VPC -kunder kan köra kod, store data, hosta webbplatser och göra allt annat de kan göra i ett vanligt private cloud, men det privata cloud:et är hostad på remote av en public cloud provider. (Det är inte alla privata moln som är värd på detta sätt.) VPC kombinerar skalbarheten och bekvämligheten med public cloud computing med dataisolering av private cloud computing.
Föreställ dig ett public cloud som en fullsatt restaurang och ett virtual private cloud som ett reserverat bord i den trånga restaurangen. Även om restaurangen är full av människor, kan ett bord med en "reserverad" skylt på den endast nås av den part som gjorde bokningen. På samma sätt är ett public cloud trångt med olika cloud kunder som har tillgång till datorresurser - men en VPC reserverar några av dessa resurser för användning av endast en kund.

Här ser ni min förklaring i bilden nedan:

![Virtual Private Cloud]({{ page.image5 }})

Referens:
- https://www.youtube.com/watch?v=HrK1UlPBkEY&ab_channel=NickChapsas
- https://docs.microsoft.com/en-us/azure/service-bus-messaging/private-link-service
- https://www.cloudflare.com/en-gb/learning/cloud/what-is-a-virtual-private-cloud/