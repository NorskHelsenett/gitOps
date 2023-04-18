---
title: "Argocd"
description: ""
lead: ""
date: 2023-04-18T09:54:41Z
lastmod: 2023-04-18T09:54:41Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "argocd-9ad614439eb7145aee82af0465ebc95f"
weight: 300
toc: true
---


ArgoCD er en gitOps operator som installeres i klusteret. Man knytter så ArgoCD opp mot et git repo der den lytter på endringer. Hvis den finne endringer, vil ArgoCD gjenspeile de endringene i klusteret. ArgoCD vil også sørge for at endringer som gjøres direkte i klusteret, alltid er i henhold til definisjonen i git. Så her er det vanskelig gå direkte i klusteret for å gjøre endringer.

Dette er også kjent som en `pull` deployment, der gitOps operatoren `henter` endringer, i motsetning til tradisjonell leveranse, der en byggepipeline `pusher` endringer inn i miljøene. På denne måten har klusteret alle hemmeligheter som kreves for å hente fra et git repo og image register. Det betyr at en byggepipeline ikke har hemmeligheter som brukes for å aksessere klusteret. Vi har nå et hermetisert miljø der færrest mulig har tilgang.

## Installer ArgoCD

```shell
kubectl create namespace argocd &&
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

{{< alert icon="👋" context="info" text="Nå må vi vente til ArgoCD har fått spunnet opp før vi kan hente admin passordet!" />}}

For å få argocd ingressen til å fungere må nginx kjøres med `--enable-ssl-passthrough`. Dette patches ved følgende kommando.

```shell
kubectl -n ingress patch daemonsets nginx-ingress-microk8s-controller \
--type=json \
-p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-ssl-passthrough"}]'
```

### Hent admin passord
{{< alert icon="ℹ️" text="Admin passordet er den enkleste måten å få tilgang til ArgoCD UI på, men følg gjerne SSO anvisningen under for å få skrudd på det istedenfor." />}}

> ℹ️ Admin passordet er den enkleste måten å få tilgang til ArgoCD UI på, men følg gjerne SSO anvisningen under for å få skrudd på det istedenfor.

Gjenta denne kommandoen frem til passordet vises, i mellomtiden er den tom.
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" 2> /dev/null | base64 -d && echo
```
## Single-sing-on (SSO) mot Gitea
{{< alert icon="ℹ️" text="Dette må gjøres etter at Gitea er ferdig provisjonert" />}}

1. Opprett ny OAUTH2 applikasjon i Gitea
- **Application Name:** ArgoCD
- **Redirect URI:**
```shell
https://argocd.local/auth/callback
```

2. Endre `clientID` `clientSecret` i `cluster/project/argocd-patches/argocd-cm.yaml`

4. Push til gitea repoet og kjør sync fra ArgoCD

5. Restart ArgoCD deployment
```shell
kubectl rollout restart deployment argocd-server -n argocd
```