---
title: 'Nyan'
date: 2019-02-11T19:27:37+10:00
weight: 5.2
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

> ℹ️ Hvis du får feilmelding på nedlasting av image der ArgoCD klager på at den ikke finner IP for git.local/gitea/nyancat:latest er den enkleste måten å få til dette å eksternet pull images via `microk8s` for å side-loade image inn i klusteret. Et annet alternativ er å restarte VM for å få inn endringene fra `/etc/hosts` filen.
```shell
microk8s ctr images pull git.local/gitea/nyancat:latest
```

# SUKSESS

![Suksess](nyancat.gif)
