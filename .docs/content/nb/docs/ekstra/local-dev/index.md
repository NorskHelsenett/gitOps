---
title: "👨‍💻 Lokal utvikling"
description: ""
lead: ""
date: 2023-04-24T21:07:02Z
lastmod: 2023-04-24T21:07:02Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "local-dev-e127a02dc68dbe7d938651f1ca15c6e9"
weight: 999
toc: true
---
## Info
Det oppsettet vi har laget nå kan lett utvides til å servere https sertifikater for lokale tjenester som kjører på samme host som klusteret.

For eksempel kan vi kjøre opp en tjeneste på port `9000` og aksessere den over `https` ved å gå til [https://dev.local](https://dev.local)

Lagre følgende til en `index.html`
```html
<!DOCTYPE html>
<html lang="no">
  <body>
    <h1>Hello World!</h1>
  </body>
</html>
```

Og kjør tjenesten ved en enkel http server som f.eks python3

```shell
python3 -m http.server 9000
```

Du kan nå aksessere denne nettsiden fra [http://localhost:9000](http://localhost:9000) (gitt at du kjører fra samme host som der `microk8s` er installert).

## Skru på HTTPS
For å skru på `https` må `local-dev` applikasjonen synkroniseres fra ArgoCD.

Denne applikasjonen laster inn tre ressurser i klusteret.

- **Ingress**: som gir oss redirect på [https://dev.local](https://dev.local), og i samarbeid `cert-manager` utsteder et sertifikat for denne URL.
- **Service**: Service som mottar trafikk fra ingressen og videresender dette til port `9000`.
- **Endpoints**: Peker på en ekstern IP og port der trafikken skal sendes. Det er viktig at klusteret har tilgang til denne ressursen da all trafikk vil traversere gjennom klusteret.

{{< alert icon="🧠" text="Endpoints kan også brukes for å tilgjengeliggjøre eksterne ressurser inn i et klusteret, f.eks en tjeneste som på et senere tidspunkt skal migreres inn i klusteret. Men som i en overgangsfase vil eksistere utenfor. Da kan interne ressurser i klusteret aksessere denne tjenesten ved å bruke den interne servicen som vil være identisk etter migrering inn i klusteret." />}}
