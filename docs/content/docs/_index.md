---
title: 'Innhold'
date: 2018-11-28T15:14:39+10:00
weight: 1
---

## GitOps

Dette er en ressursside for hvordan komme i gang med gitOps, enten du skal ta i bruk noe profesjonelt eller bare sette opp noe enkelt hjemme.

gitOps er en pull metodikk, der en operator som installert i hvert miljø vil gå mot et git repo for å lytte på endringer. gitOps operatoren vil alltid sørge for at kjøretidsmiljøet er i henhold til spesifikasjonen som ligger på git. Endres git, vil operatoren gjenspeile det i klusteret. Kjent som en leveranse. Hvis noen endrer på ressursene i klusteret, vil gitOps reversere det slik at det igjen er likt det som er definert.

Siden gitOps jobber et pull metodikken, vil det si at det er klusteret som har tilgang på de nødvendige hemmelighetene for å få tilgang til git eller image repo som dockerhub. Dermed kan klusteret være helt hermetisert og isolert, noe som er sikrere enn en tradisjonell push metodikk der en byggelinje f.eks pusher artifaktene til miljøene. I de tilfellene må byggelinjen har de nødvendige tilgangene for å kunne kontakte miljøene.

### Oppsett
I dette oppsett inngår følgende portefølje av produkter:

Produkt | URL | Beskrivelse
---:|---|---
ArgoCD | [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) | gitOps operatoren vår, og verdens søteste logo.
Gitea | [Gitea](https://gitea.io/en-us/) | on-prem github alternativ
DroneCI | [DroneCI](https://www.drone.io) | bygg agenter for bygging av artifakter og produkter
Nyan | [NyanCat](https://github.com/cristurm/nyan-cat) | nyancat - brukes for å teste ingress, sertifikater og byggelinje
Cert-Manager | [Cert-Manager](https://cert-manager.io) | brukes for å utstede selv signerte sertifikater
NGINX | [NGINX](https://www.nginx.com) | ingress kontrolleren vår, på en måte en reverse-proxy
Multipass |[ Multipass](http://multipass.run) | brukes for å opprette en lokal VM, på mange måter veldig likt WSL
microk8s |[ microk8s](http://microk8s.io) | et produkt for å provisjonere et kubernetes kluster. Et av mange gode alternativer