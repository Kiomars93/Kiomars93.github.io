---
layout: post
title:  "Kap 3 - 2021-09-13 Containrar och orkestrering"
date:   2021-09-13 09:31:04 +0200
categories: jekyll update

---


# Det som jag har installerat lokalt / Github:

Jag hade docker desktop installerat sen innan för att se mina images och containrar. Github desktop
för git hantering, fork och etc.

# Beskrivning om hur jag fick applikationen att köras i en container:

I vårt fall så hade vi ett färdigt "Hello World" repo som jag forkade ner. Vill man ha en ny från scratch så får man köra på antingen starta ett nytt asp projekt via
antingen visual code eller visual studio direkt. Annars kan man göra det via kommandon genom att:

``` cmd

På det viset skapar vi filerna i sig:

mkdir SimpleWebHalloWorld
cd SimpleWebHalloWorld
mkdir src
cd src
mkdir AspMVC
cd AspMVC

Därefter skapar vi programmet genom att:

dotnet new MVC

```

``` cmd

Som sagt i vårt fall kör vi det existerande repon 
och skapar en ny docker fil genom att skriva:

code dockerFile 

```
Därefter fyller vi vår dockerFile med det som behövs för att kunna skicka upp det vi vill till molnet.

``` cmd
Sedan skapar vi en image genom att skriva:

docker build -t mydockerapp .

Därefter kunna koppla vår lokala port mot docker porten 
(I vårt exempel så är vårt lokala 8080 och dockers 80):

docker run -d -p 8080:80 --name MyNetApp myapp

Vi kan se om vårt app är dockerized och fungerande genom att skriva:

docker ps

ovanstående visar alla våra aktiva containers
```

# Beskrivning av min docker fil:

``` csharp
// Första raden här är ju själva baseimagen där vi vill ha vår dockerfil
FROM mcr.microsoft.com/dotnet/sdk:3.1 AS base

// vilken directory den kopierar ifrån
WORKDIR /app
COPY *.csproj ./
// Kommandot nedan använder NuGet för att återställa dependencies såväl som projektspecifika verktyg
// som anges i projektfilen.
RUN dotnet restore
COPY . ./
// Det här tillgänglig gör en port flr tjänster utanför Docker eller för Docker containrar som inte är ansluta till
// containers nätverk.
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:3.1
WORKDIR /app
COPY --from=base /app/out .
// Med ENTRYPOINT -instruktionen kan du konfigurera en contrainer som körs som en körbar.
ENTRYPOINT ["dotnet", "SimpleWebHalloWorld.dll"]

```
# Beskrivning av min github pipeline:
Min docker build and push pipeline består av flera steg:

1. Först så skapar det själva jobbet så jag har angett. Mitt jobbnamn är docker-build-and-push.
2. Därefter så checkout den all kod till github repon.
3. Loggar in via mitt user och pass och mer detalj kring inloggningen har jag skrivit om nedan.
4. Bygger och pushar allt

Därefter har jag kopplat mitt repo med paket inställningar där containers källkod kommer att lagras på.
Länken till detta är:

"org.opencontainers.image.source = &quot;https://github.com/Kiomars93/SimpleWebHalloWorld&quot;"
Gjorde detta därefter synligt för att kunna dra detta till min container om jag vill det.

Detta är en kortbeskrivning min workflow som jag har lagt till min YAML fil.
``` yaml

# Detta är mitt grundläggande workflow som jag har döpt till docker pipeline
name: docker pipeline

# mitt on event är upplagd på det sättet att det skapas ett nytt workflow asap jag ändrar något och gör en ny commit
on: [push]

jobs:
  # Jag har ett jobb här nedan och har döpt den till docker-build-and-push
  docker-build-and-push:
    # Min runnertyp som mitt workflow kommer att köras på
    runs-on: ubuntu-latest
    # steps representerar ett antal uppgifter som kommer att köras
    steps:
    # namn på min checkout och uses under det är all kod så jag skickar över till min github-workspace
    - name: Checkout code
      uses: actions/checkout@v2.3.4
    # Loggade in via mitt github user och pass, Mitt token har jag döpt till "MYSECRET_DOCKER".
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v1.10.0
      with:
         registry: ghcr.io
         username: ${{ github.actor }};
         password: ${{ secrets.MYSECRET_DOCKER }}
    - name: Build and push
      id: docker_build
      uses: docker/build-push-action@v2.7.0
      with:
        push: true
        context: ${{ env.working-directory }}
        # detta är min repo directory och allt måste vara i små bokstäver
        tags: |
          ghcr.io/kiomars93/simplewebhalloworld:latest
          ghcr.io/kiomars93/simplewebhalloworld:${{ github.run_number }}
```


# Mitt sätt att göra min token secret:

Det gjorde jag genom att:
1. Välja min profil ikon och därefter setting.
2. Developer Settings taben.
3. Valde PAT taben.
4. Genererade en 7 dagars valid token
5. Därefter gick jag i mitt forkade repo
6. Valde settings taben
7. Valde "New repository secret"
8. Döpte det till MYSECRET_DOCKER som ni såg tidigare på min "docker pipeline" 
workflow och la till mitt token value och sparade därefter.

