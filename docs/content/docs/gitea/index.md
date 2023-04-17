---
title: 'Gitea & Drone CI'
date: 2019-02-11T19:30:08+10:00
draft: false
weight: 5
summary: |
  Gitea er et verktøy for å hoste sine egne git repositories.
  DroneCI er et byggelinje verktøy for å bygge applikasjoner etter definisjoner.
  Disse to spiller på lag, slik at hver commit til Gitea starter et bygg i DroneCI.

---
Gitea er et verktøy for å hoste sine egne git repositories.

DroneCI er et byggelinje verktøy for å bygge applikasjoner etter definisjoner.

Disse to spiller på lag, slik at hver commit til Gitea starter et bygg i DroneCI.

Når du nå åpner ArgoCD i nettleseren vil du se en app som heter `cluster` og som er ute av sync. Hvis du så trykker på synkroniser vil du se at den oppretter en Gitea, Drone server og Drone runner instans for oss.


```bash
brukernavn: gitea
passord: gitops
```

## Importer (fork) GitHub repoet

![clone-github-repo](clone-github-repo.png)

![gitea-migrate-git](gitea-migrate-git.png)

![gitea-migrate-choose](gitea-migrate-choose.png)

![gitea-migrate-import-window](gitea-migrate-import-window.png)

### Lag OAuth2 applikasjon i Gitea

![oauth settings](oauth.png)

Lag en OAuth2 applikasjon med følgende innstillinger:
```bash
Application Name: DroneCI
Callback URI: https://drone.local/login
```

Kopier `Client ID` og `Client Secret` og bruk de som override i ArgoCD for DroneCI etter installasjon i steget under.
![Enable OAUTH](gitea-drone-oauth2.png)

![Input DroneCI - gitea client-id and Client-Secret](argocd-drone-secret.png)

# DroneCI

## Overskriv parametere
Gå til [Drone.local](https://drone.local) for å logge inn.

![Velkommen](drone-welcome.png)

Du vil nå bli sendt til Gitea for autentisering før du kommer tilbake til DroneCI for registrering. `gitea` brukeren er allerede registert som admin i DroneCI fra JWT token til Gitea, så her kan du skrive hva som helst i feltene

![DroneCI Registrering](drone-register.png)

Når du nå synkroniserer prosjektet vil ArgoCD installere DroneCI server og runner.

![Synkroniser](drone-sync.png)

*Hvis det fortsatt er tomt, sørg for at `Active Only` ikke skrudd på*.

Aktiver repoet, og legg inn i disse to hemmelighetene:

Variabel | Verdi
---|--:
registry_username | gitea
registry_password | gitops

![Drone Secret](drone-secret.png)

Nå er det klart for å starte et nytt bygg.

![Drone Dashboard](drone-dashboard.png)