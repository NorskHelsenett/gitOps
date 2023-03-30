---
title: 'Sertifikater'
date: 2019-02-11T19:27:37+10:00
weight: 7
---
Hvis du har et eget domene kan du knytte opp cert-manager og nginx mot f.eks Let's Encrypt. Men i dette eksemplet skal vi se på et internt kluster. Og for å få kjørt tls kryptering mellom tjenestene, og ut av klusteret, må vi lage vårt eget sertifikat som cert-manager bruker for å utstede sertifikater for alle tjenestene.

## Opprett et nytt CA sertifikat

Det opprettes automatisk eget CA og sertifikater for hver enkelt tjeneste. Det som nå må gjøres er å manuelt godkjenne CA sertifikatet på egen maskin og på den maskinen der klusteret kjører. Dette for å kunne få tilgang til tjenestene uten å måtte si at man godtar koblingen selv om den ikke er "sikker".

I noen tilfeller som i DroneCI så gjøres det her et unntak, da byggeagentene ikke automatisk får tilgang til CA sertifikatene, og det i sånn måte må importeres som en del av byggepipeline. Av praktiske, og av læringsøymed er disse stegene heller utelatt.

### Importere sertifikatene på egen maskin og i kluster

```shell
# Hent ut nåværende issuer sertifikat.
kubectl get secrets -n cert-manager self-signed-ca -o json | jq -r '.data["tls.crt"]' | base64 -d > issuer.crt

# Installer sertifikat i lokal cert-store
sudo cp issuer.crt /usr/local/share/ca-certificates/issuer.crt
sudo update-ca-certificates

# Test om sertifikatet er gyldig. Denne kommandoen skal skrive ut HTML kode om den er TLS er gyldig.
curl -L argocd.local
```

### Importere CA sertifikatet inn i podene

For at gitea og droneci skal kunne kommunisere over en sikker TLS forbindelse, må man legge inn `issuer.crt` filen i hver eneste pod. Dette skal vi gjøre ved hjelp av en configmap, og er et stykke manuelt arbeid.

`ca.crt` er tilgjengelig som en hemmelighet i de `namespace` som har spurt etter et sertifikat. Hemmeligheten som lages består av 3 datafelt, `ca.crt`, `tls.crt` og `tls.key`.

- `ca.crt`
  - Dette feltet er den offentlige nøkkelen til issuer sertifikatet som vi får fra klusteret.
- `tls.crt`
  - Dette er den offentlige nøkkelen for selve tjenesten.
- `tls.key`
  - Dette er privatnøkkelen for tjenesten, (leaf sertifikatet) f.eks det sertifikatet som gir oss https://drone.local

Det vi må gjøre hvis vi ønsker en tjeneste som kommuniserer med en annen tjeneste over `HTTPS` i klusteret, er å importere `ca.crt` inn i noden. Det er forskjellig for hvert HELM chart, men forhåpentligvis er det en god oppskrift på hvordan man kan gjøre det. Sluttresultatet manifestet skal bli som følger:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: drone
  namespace: drone
spec:
  containers:
    - image:
      volumeMounts:
        - mountPath: /etc/ssl/certs/ca.crt
          name: cluster-ca
          readOnly: true
          subPath: ca.crt #denne er VIKTIG. Hvis ikke overskrives hele mappen. Dette er også verdien fra hemmeligheten vi ønsker å hente ut.
  volumes:
    - secret:
        secretName: drone-tls-secret
      name: cluster-ca
```