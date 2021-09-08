---
layout: post
title:  "Moln-priser"
date:   2021-09-07 09:31:04 +0200
image1:  /Image/aws1.PNG
image2:  /Image/aws2.PNG
image3:  /Image/azure.PNG
image4:  /Image/azure2.PNG
image5:  /Image/googlecloud.PNG
---

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