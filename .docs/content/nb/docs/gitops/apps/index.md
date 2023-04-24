---
title: "Apps of Apps"
description: ""
lead: ""
date: 2023-04-18T09:55:09Z
lastmod: 2023-04-18T09:55:09Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "apps-1d6ac2c902703ff5a33465be96fd3fd8"
weight: 400
toc: true
---

Siden vi jobber med gitOps skal vi nå gå mot et ekte git repo som grunnlag for hva ArgoCD skal gjøre fremover.

Installer `app-of-apps` i klusteret
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/NorskHelsenett/gitOps/main/cluster/app-of-apps.yml
```

Følg med på progresjonen ved å kjøre `kubectl get apps -n argocd`. Når app-of-apps står som `Healthy` er det klart for å fortsette til neste steg.
```shell
kubectl get apps -n argocd

NAME          SYNC STATUS   HEALTH STATUS
app-of-apps    Synced        Healthy
argocd-sso     Synced        Healthy
drone          Synced        OutOfSync
drone-runner   Synced        OutOfSync
gitea          Synced        Healthy
nyan           OutOfSync     Missing
tooling        Synced        Healthy
```

Når `app-of-apps`, `gitea` og `tooling` viser `Healthy` har vi de viktigste komponentene installert. 

## Aksessere ArgoCD

Før du kan gå til ArgoCD nettsiden må du installere root sertifikatet som lages av `app-of-apps`.

Dette gjør du ved å følge stegene på [Sertifikater](../sertifikater), og kom tilbake hit etter at det er gjort.

1. Installer root sertifikatet på egen maskin
2. Legg til `ip` og `hostname` i `hosts` filen på egen maskin

Når alle disse stegene er utført, vil du kunne nå argocd gjennom [https://argocd.local](https://argocd.local).