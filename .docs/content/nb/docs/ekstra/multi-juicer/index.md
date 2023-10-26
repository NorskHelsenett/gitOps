---
title: "🍹 Multi Juicer"
description: ""
lead: ""
date: 2023-04-24T21:07:02Z
lastmod: 2023-04-24T21:07:02Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "multi-juicer-e127a02dc68dbe7d938651f1ca15c6e9"
weight: 1023
toc: true
---

MultiJuicer er et verktøy for å provisjonere og hoste CTF ved bruk av OWASP JuiceShop sin sårbare opplærings portal for pentesting og applikasjonssikkerhet.

{{< alert icon="ℹ️" context="info" text="MultiJuicer er tilgjengelig på github, gi det en stjerne 🌟" />}}

---

🎯 SCOREBOARD: [https://ctf.local/balancer/score-board/](https://ctf.local/balancer/score-board/)

🌐 URL: [https://ctf.local](https://ctf.local)

📦 github: [https://github.com/juice-shop/multi-juicer](https://github.com/juice-shop/multi-juicer)

---

## Admin Passord
Admin passord hentes ut ved:

```bash
kubectl get secrets juice-balancer-secret -o=jsonpath='{.data.adminPassword}' -n juiceshop | base64 --decode && echo
```

## NetworkPolicy
Det er lagt på en egen NetworkPolicy rundt JuiceShop, for å sikre at den kun kan kommunisere med de tjenestene den har behov for å kommunisere med. Default regel er deny på inngående og utgående.

![NetworkPolicy schema drawing](networkPolicy.png)
