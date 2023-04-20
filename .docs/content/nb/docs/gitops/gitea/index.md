---
title: "Gitea"
description: ""
lead: ""
date: 2023-04-18T09:55:22Z
lastmod: 2023-04-18T09:55:22Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "gitea-ac03be39b053375fee6b31cffe37bde3"
weight: 500
toc: true
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

```shell
https://github.com/NorskHelsenett/gitOps.git
```

![clone-github-repo](clone-github-repo.png)

![gitea-migrate-git](gitea-migrate-git.png)

![gitea-migrate-choose](gitea-migrate-choose.png)

![gitea-migrate-import-window](gitea-migrate-import-window.png)

## OAuth2 i Gitea

Lag en OAuth2 applikasjon med følgende innstillinger:
```bash
Application Name: DroneCI
Callback URI: https://drone.local/login
```

![oauth settings](oauth.png)



Kopier `Client ID` og `Client Secret` og commit de til `cluster/project/drone/drone-secrets.yml`.
![Enable OAUTH](gitea-drone-oauth2.png)

Lagre disse verdiene fra Gitea til `cluster/project/drone/drone-secrets.yml` ved en commit.
![Commit drone secret to gitea](gitea-drone-secret-commit.png)

Trykk på **REFRESH** i ArgoCD og hemmeligheten vil automatisk oppdateres
![synchronise changes in argocd](drone-secret-argocd.png)

**RESTART** DroneCI Deploymentet etter at hemmeligheten er synkroniser, for at DroneCI skal ta det i bruk.
![Restart drone deployment](drone-restart.png)

## DroneCI
### Installasjon
Gå til [Drone.local](https://drone.local) for å logge inn.

![Velkommen](drone-welcome.png)

Du vil nå bli sendt til Gitea for autentisering før du kommer tilbake til DroneCI for registrering. `gitea` brukeren er allerede registert som admin i DroneCI fra JWT token til Gitea, så her kan du skrive hva som helst i feltene

![DroneCI Registrering](drone-register.png)

Når du nå synkroniserer prosjektet vil ArgoCD installere DroneCI server og runner.

![Synkroniser](drone-sync.png)

*Hvis det fortsatt er tomt, sørg for at `Active Only` ikke skrudd på*.

### Hemmeligheter
Aktiver repoet, og legg inn i disse to hemmelighetene

`Settings > Organization (Secrets)`

Variabel | Verdi
---|--:
registry_username | gitea
registry_password | gitops

![Drone Secret](drone-org-secret.png)

Nå er det klart for å starte et nytt bygg.

![Drone Dashboard](drone-dashboard.png)

{{< alert icon="👨‍💻" context="info" text="På grunn av automatisert CI/CD så risikerer man en uendelig commit loop mellom gitea og drone! En commit til gitea vil trigge et bygg i drone, som så vil pushe en git commit tilbake til gitea 👉👈" />}}
