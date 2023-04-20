---
title: "Nyan"
description: ""
lead: ""
date: 2023-04-18T09:55:37Z
lastmod: 2023-04-18T09:55:37Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "nyan-bbecba0ede3e4d9c2865454069303294"
weight: 600
toc: true
---


![NyanCat](nyancat.png)

**Sørg for følgende:**
- Du har klonet repoet inn til git.local fra github
- DroneCI har synkronisert repoet fra git.local
- Hemmelighetene `registry_username` og `registry_password` er lagt til i DroneCI
- DroneCI har bygd nyancat

Målet er å få https://nyan.local opp og kjøre. Hvis du prøver å gå til denne siden nå, vil du få en `This website is not available`.

![This website is not available](nyan.local-unavailable.png)

Gå tilbake til ArgoCD og synkroniser applikasjonen,
![Sync Nyancat](argocd-sync-nyan-app.png)

{{< alert icon="ℹ️" text="Hvis du får feilmelding på nedlasting av image der ArgoCD klager på at den ikke finner IP for git.local/gitea/nyancat:latest er den enkleste måten å få til dette å eksternet pull images via microk8s for å side-loade image inn i klusteret. Et annet alternativ er å restarte VM for å få inn endringene fra /etc/hosts filen." />}}

```shell
microk8s ctr images pull git.local/gitea/nyancat:latest
```
{{< alert icon="ℹ️" context="success" text="Hvis du har fulgt guiden riktig så vil du nå få opp nyancat på nyan.local 👏 GRATULERER!! 👏" />}}
