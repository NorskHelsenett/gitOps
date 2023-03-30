---
title: 'App of apps'
date: 2019-02-11T19:27:37+10:00
weight: 4
---

Siden vi jobber med gitOps skal vi nå gå mot et ekte git repo som grunnlag for hva ArgoCD skal gjøre fremover.

Installer `app-of-apps` i klusteret
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/jonasbg/gitOps-intro/static-site/cluster/app-of-apps.yml
```

Følg med på progresjonen ved å kjøre `kubectl get apps -n argocd`. Når app-of-apps står som `Healthy` er det klart for å fortsette til neste steg.
```shell
kubectl get apps -n argocd

NAME          SYNC STATUS   HEALTH STATUS
app-of-apps   Synced        Healthy
cluster       OutOfSync     Healthy
```

## Aksessere ArgoCD

Før du kan gå til ArgoCD nettsiden må du installere root sertifikatet som lages av `app-of-apps`.

Dette gjør du ved å følge stegene på [Sertifikater](/sertifikat)

1. Installer root sertifikatet på egen maskin
2. Legg til `ip` og `hostname` i `hosts` filen på egen maskin

Når alle disse stegene er utført, vil du kunne nå argocd gjennom https://argocd.local.