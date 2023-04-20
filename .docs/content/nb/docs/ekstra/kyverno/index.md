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
{{< alert icon="ℹ️" text="Kyverno tilbyr en best-practices profil som kan installeres. På den måten for du en tidlig starthjelp på en sikker utforming av dine tjenester som skal kjøre i et kluster." />}}

[Best Practices](https://github.com/kyverno/policies/best-practices)

**Dette repoet inneholder alle disse regelsettene.**