---
title: "Kyverno"
description: ""
lead: ""
date: 2023-04-18T09:58:09Z
lastmod: 2023-04-18T09:58:09Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "kyverno-26d49d616d1178254ad443073584be7a"
weight: 999
toc: true
---

Kyverno brukes for å sikre at ressurser i klusteret er i henhold til en profil som er definert basert på sikkerhet.

I vårt eksempel skal vi bruke kyverno for å sikre at klusteret kun kjøre signerte ressurser i de `namespace` som vi definerer.

## Best Practises
{{< alert icon="ℹ️" text="Kyverno tilbyr en best-practices profil som kan installeres. På den måten for du en tidlig starthjelp på en sikker utforming av dine tjenester som skal kjøre i et kluster. Dette repoet inneholder et utvalgt sett av dette regelsettet." />}}

Ler mer om dette regelsettet: [Best Practices](https://github.com/kyverno/policies/best-practices)

Følgende namespaces er ekskludert for dette repoet for enkelthets skyld, slik at det lettere å fokusere på de namespace som man faktisk jobber med:
- argocd
- cert-manager
- default
- drone
- gitea
- ingress
- kube-node-lease
- kube-public
- kube-system
- kyverno
- nyan
- vault

## Rapportering
```shell
kubectl get policyreport -A
```
Gir følgende resultat
```shell
NAMESPACE   NAME                               PASS   FAIL   WARN   ERROR   SKIP   AGE
nyan        cpol-disallow-empty-ingress-host   1      0      0      0       0      41m
nyan        cpol-restrict-external-ips         1      0      0      0       0      54m
nyan        cpol-check-deprecated-apis         0      0      0      0       1      54m
```

```shell
kubectl get clusterpolicyreport
```