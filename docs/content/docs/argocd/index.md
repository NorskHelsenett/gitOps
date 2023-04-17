---
title: 'ArgoCD'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 3
---

ArgoCD er en gitOps operator som installeres i klusteret. Man knytter så ArgoCD opp mot et git repo der den lytter på endringer. Hvis den finne endringer, vil ArgoCD gjenspeile de endringene i klusteret. ArgoCD vil også sørge for at endringer som gjøres direkte i klusteret, alltid er i henhold til definisjonen i git. Så her er det vanskelig gå direkte i klusteret for å gjøre endringer.

Dette er også kjent som en `pull` deployment, der gitOps operatoren `henter` endringer, i motsetning til tradisjonell leveranse, der en byggepipeline `pusher` endringer inn i miljøene. På denne måten har klusteret alle hemmeligheter som kreves for å hente fra et git repo og image register. Det betyr at en byggepipeline ikke har hemmeligheter som brukes for å aksessere klusteret. Vi har nå et hermetisert miljø der færrest mulig har tilgang.

## Installer ArgoCD

```shell
kubectl create namespace argocd &&
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> 👋  **Ta deg en kopp kaffe**: Nå må vi vente til ArgoCD har fått spunnet opp før vi kan hente admin passordet!

### Hent admin passord
> ℹ️ Admin passordet er den enkleste måten å få tilgang til ArgoCD UI på, men følg gjerne SSO anvisningen under for å få skrudd på det istedenfor.

Gjenta denne kommandoen frem til passordet vises, i mellomtiden er den tom.
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" 2> /dev/null | base64 -d && echo
```
## Single-sing-on (SSO) mot Gitea
> ℹ️ Gjør dette etter at Gitea er satt opp

1. Opprett ny OAUTH2 applikasjon i Gitea
- **Application Name:** ArgoCD
- **Redirect URI:** https://argocd.local/auth/callback

2. Endre `clientID` `clientSecret` i `cluster/project/argocd-patches/argocd-cm.yaml`

3. Kjør følgende kommandoer fra rootmappen
```shell
kubectl patch configmap argocd-cm -n argocd --patch-file cluster/project/argocd-patch/argocd-cm-patch.yml
kubectl patch configmap argocd-rbac-cm -n argocd --patch-file cluster/project/argocd-patch/argocd-rbac-cm-patch.yml
kubectl patch deployment argocd-server -n argocd --patch-file cluster/project/argocd-patch/argocd-patch.yml
```