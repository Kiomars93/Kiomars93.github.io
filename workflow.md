---
layout: page
title: Workflow
permalink: /workflow/
---

# Beskrivning av CI pipline

    CI kan hjälpa dig att hålla dig till ditt teams kvalitetsstandarder genom att köra tester och 
    rapportera resultaten på GitHub. CI -verktyg kör builds och tester, utlösta av commits. 
    Resultaten skickas tillbaka till GitHub i pull request. Detta minskar kontextbyte för utvecklare och förbättrar 
    konsistensen för testning. 
    Målet är färre buggar i produktionen och snabbare feedback under utvecklingen.

     Val kring CI som fungerar bäst för ditt projekt beror på många faktorer, inklusive:

     * Programmeringsspråk och applikationsarkitektur
     * Operativsystem och webbläsare du planerar att stödja
     * Ditt teams erfarenhet och kompetens
     * Skalningsfunktioner och tillväxtplaner
     * Geografisk fördelning av beroende system och de människor som använder dem
     * Förpacknings- och leveransmål
     
     Referens: https://github.com/Kiomars93/github-actions-for-ci/issues/1

# Implementation av en CI pipeline i Github actions på vårt spacepark-team7 projekt och förklaring av dessa stegvis:

    Ja, har skapat en workflow på vår spacepark-team7 repo och kallat det för Run-Flow.

    Dessa steg har jag gått igenom:

    Workflows
    Workflow är ett automatiserat procedur som du lägger till i ditt repo. Workflow består av ett eller 
    flera jobs och kan schemaläggas eller utlösas av ett event. Workflow kan användas för att build, 
    test, package, release eller deploy ett projekt på GitHub.

    Events
    En event är en specifik aktivitet som utlöser ett workflow. Till exempel kan aktivitet utgå från GitHub
    när någon pushar en commit till ett repo eller när en issue eller pull request skapas. 
    Du kan också använda webbhooken för repoutskick för att utlösa ett workflow när en extern event inträffar. 
    En fullständig lista över events som kan användas för att utlösa workflows, se Events som utlöser workflows.

    Jobs
    Ett jobb är en uppsättning steg som utförs på samma runner. Som standard kommer ett workflow med flera 
    jobb att köra dessa jobb parallellt. Du kan också konfigurera ett workflow för att köra jobb i följd. 
    Till exempel kan ett workflow ha två sekventiella jobb som bygger och testar kod, där testjobbet är 
    beroende av byggjobbets status. Om byggjobbet misslyckas körs inte testjobbet.

    Steps
    Ett steg är en individuell uppgift som kan köra kommandon i ett jobb. Ett steg kan vara antingen 
    en action eller ett shell command. Varje steg i ett jobb utförs på samma runner, så att åtgärderna 
    i det jobbet kan dela data med varandra.

    Actions
    Åtgärder är fristående kommandon som kombineras till steg för att skapa ett jobb. 
    Actions är den minsta bärbara byggstenen i ett workflow. Du kan skapa dina egna 
    actions eller använda actions som skapats av GitHub -gemenskapen. Om du vill använda en åtgärd i 
    ett workflow måste du inkludera den som ett steg.

    Runners
    En runner är en server som har GitHub Actions -runner applikation installerat. 
    Du kan använda en runner som hostas av GitHub, eller så kan du vara värd för din egen. 
    En runner lyssnar efter tillgängliga jobb, kör ett jobb i taget och rapporterar framsteg, 
    loggar och resultat tillbaka till GitHub. GitHub-värdrunners är baserade på 
    Ubuntu Linux, Microsoft Windows och macOS, och varje jobb i ett workflow körs i en ny virtuell miljö. 
    För information om GitHub-värdrunners, se "Om GitHub-värdrunners." 
    Om du behöver ett annat operativsystem eller behöver en specifik hårdvarukonfiguration kan 
    du vara värd för dina egna runners. Mer information om runners med egen värd finns i "Värd för dina egna runners".

    Referens: https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions
    
# Beskrivning av min GitHub action workflow YAML fil: 

``` yaml
# Detta är mitt grundläggande workflow som jag har döpt till CI

name: CI

# Det som kontrollerar om min workflow ska köras är "on" nedan:
on:
  # Utlöser arbetsflödet vid push- eller pull -begärningshändelser men bara för main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  # Låter dig köra detta workflow manuellt från fliken Actions
  workflow_dispatch:

# En workflow består av ett eller flera jobs som kan köras i följd eller parallellt
jobs:
  # Detta wrokflow innehåller ett enda jobb som kallas "build"
  build:
    # Den typ av runner som jobbet kommer att köras på
    runs-on: windows-latest

    # Stegen representerar en sekvens av uppgifter som kommer att utföras som en del av jobbet
    steps:
      # Checks-out ditt repo under $ GITHUB_WORKSPACE, så att ditt job kan ha acess till det
      - uses: actions/checkout@v2

      # Kör ett enda kommando med hjälp av runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Kör en uppsättning kommandon med hjälp av runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.

```

