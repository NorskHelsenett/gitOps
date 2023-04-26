---
title: "🔏 Signering"
description: ""
lead: ""
date: 2023-04-18T09:58:37Z
lastmod: 2023-04-18T09:58:37Z
draft: false
images: []
menu:
  docs:
    parent: ""
    identifier: "signering-ebeb195a06126ce9f6c881381dd6aee3"
weight: 850
toc: true
---

Signering er for å lage et kryptografisk stempel på autensitet og integritet.

**Autensitet:**
Ved å signere kan andre verifisere på opphavet

**Integritet:**
Ved å signere kan man sikre at det som blir levert, er det som er tiltenkt levert, uten endringer.

**Krav:**
Ved å sette krav til signering kan man også stenge for at uønskede aktører kan kjøre opp tjenester som ikke er signert.

Produktet vi skal vise frem heter [Cosign fra Sigstore](https://docs.sigstore.dev/) og det er en enkel måte å signere images, artifakter, BOM osv...

## Installasjon
Oppdatert guide er alltid tilgjengelig på [Cosign Installation](https://docs.sigstore.dev/cosign/installation/)

### GO
```shell
go install github.com/sigstore/cosign/v2/cmd/cosign@latest
```

### Homebrew
```shell
brew install cosign
```

### Binary
```shell
ARCH=$(dpkg --print-architecture)
wget "https://github.com/sigstore/cosign/releases/download/v2.0.2/cosign-linux-$ARCH"
sudo mv cosign-linux-$ARCH /usr/local/bin/cosign
sudo chmod +x /usr/local/bin/cosign
```

## Signeringsmetodikker

### Nøkkelpar
Her brukes nøkkelpar (private/public key) for å signere og verifisere images.

Signering med nøkkelpar er den enkleste formen for signering, og kan enkelt integreres i ethvert team.

```shell
cosign generate-key-pair
cosign sign --key cosign.key alpine/curl
```

Privat nøkkel må sikres på et forsvarlig sted.

### Key Management System (KMS)
Her benytter man en ekstern aktør for å holde på og sikre privatnøkkelen. Et bra KMS vil aldri kunne gi fra seg en privatnøkkel, det eneste man får tilgang på er den offentlige nøkkelen.

All signering skjer i selve KMS. Ved signering av images er det digest verdien som blir sendt til KMS for signering.

I dette eksemplet skal vi bruke [HashiCorp Vault](/vault/) som KMS tilbyder. Dette forutsetter at transit engine er skrudd på ihht [dokumentasjonen](../vault/#transit-engine-og-signering).

```shell
cosign generate-key-pair --kms hashivault://signature

cosign sign --kms hashivault://signature git.local/gitea/nyancat
```

### Sertifikater
Sertifikat signering og sertifisering er best brukt i større organisasjoner, der en organisasjon kan utstede ut fra et eget sertifikat, et nytt sertifikat for hvert enkelt team. Ut fra dette sertifikatet som inneholder tre komponenter, en privatnøkkel, en offentlig nøkkel for sertifikatet og den offentlige nøkkelen til utsteder, kan man generere en cosign nøkkel for signering. En organisasjon vil så sette krav til at alle signeringer referer til denne tillitskjeden av sertifikater, og at alle har opphav fra organisasjons sertifikatet.

Se [Hvordan lage en sertifikat kjede lokalt med OpenSSL](#openssl-sertifikater) for veiledning på hvordan man lager en gyldig sertifikat kjede lokalt.

### OIDC
**O**pen **ID** **C**onnect

---

## Veiledere
### Bruke clusterets sertifikater

{{< alert icon="ℹ️" context="danger" text="Dette er for fremtidig notis og er ikke ferdig implementert!" />}}

```shell
kubectl get secrets -n kyverno kyverno-tls-secret -o json | jq -r '.data["tls.crt"]' | base64 -d > cert.crt
kubectl get secrets -n kyverno kyverno-tls-secret -o json | jq -r '.data["tls.key"]' | base64 -d > cert.key
kubectl get secrets -n kyverno kyverno-tls-secret -o json | jq -r '.data["ca.crt"]' | base64 -d > ca.crt
cosign import-key-pair --key cert.key
cosign sign --key import-cosign.key --yes --cert cert.crt --cert-chain ca.crt git.local/gitea/nyancat
```

### HashiCorp Vault PKI
HashiCorp Vault kan brukes som en PKI tilbyder for utstedelse av sertifikater og sertifikat kjede. Dette er veiledning for å sette opp Vault som en autorisert sertifikat tilbyder.

### OpenSSL sertifikater

For å ta i bruk sertifikater må man lage en sertifikat kjede som er gyldig. Dette kan du gjøre med følgende fremgangsmetode:

1. Generate a private key for Root certificate

```shell
# Husk å bruk passord her, ellers funker det ikke!
openssl genrsa -des3 -out rootCA.key 2048
```

2. Generate Root certificate

```shell
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.crt
```

in Certificate generation set following values
```
C = IN, ST = DEL, L = DEL, O = example.com, OU = sigstore, CN = sigstore, emailAddress = foo@example.com
```

3. Generate Private key for Intermediate certificate

```shell
openssl genrsa -out intermediateCA.key 2048
```

4. Generate CSR for Intermediate certificate

```shell
openssl req -new -key intermediateCA.key -out intermediateCA.csr
```

in Certificate generation set following values
```
C = IN, ST = DEL, L = DEL, O = example.com, OU = sigstore-sub, CN = sigstore-sub, emailAddress = foo@example.com
```

5. Create intermediate certificate config file by name "intermediateConfigFile" having content

```
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
```


6. Create intermediate certificate

```shell
openssl x509 -req -in intermediateCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -CAserial intermediateca.srl -out intermediateCA.crt -days 1825 -sha256 -extfile intermediateConfigFile
```

7. Create Private key for leaf certificate

```shell
openssl genrsa -out leafCA.key 2048
```

8. Create CSR for Leaf certificate

```shell
openssl req -new -key leafCA.key -out leafCA.csr
```

in certificate generation set following values
```
C = IN, ST = DEL, L = DEL, O = example.com, OU = sigstore-leaf, CN = sigstore-leaf, emailAddress = foo@example.com
```

9. Create Leaf certificate config file by name "leafConfigFile" having content

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage=codeSigning
subjectAltName=email:copy
```


10. Create Leaf certificate

```shell
openssl x509 -req -in leafCA.csr -CA intermediateCA.crt -CAkey intermediateCA.key -CAcreateserial -CAserial leafca.srl -out leafCA.crt -days 1825 -sha256 -extfile leafConfigFile
```



11. Generate Certificate chain by concatinating Intermediate certificate and Root certificate

```shell
cat intermediateCA.crt rootCA.crt > certChain.crt
```



12. Import key

```shell
cosign import-key-pair --key leafCA.key
```



13. Sign cosign

```shell
# Fungerer bare med verifisering
cosign sign --key import-cosign.key git.local/gitea/nyancat

cosign sign --key import-cosign.key --yes --cert leafCA.crt --cert-chain intermediateCA.crt git.local/gitea/nyancat
```

14. Verify signature
```shell
cosign verify git.local/gitea/nyancat --cert leafCA.crt --cert-chain certChain.crt | jq .

cosign verify git.local/gitea/nyancat --cert leafCA.crt --cert-chain intermediateCA.crt | jq
```

15. Check status
```shell
kubectl get AdmissionReport -A
```

## Kyverno

### Key-pair
Lagre følgende clusterPolicy til `kyverno` namespace.

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image
spec:
  validationFailureAction: Enforce
  rules:
    - name: verify-signature
      match:
        any:
        - resources:
            kinds:
              - Pod
      - resources:
          namespaces:
            - nyan
      verifyImages:
      - imageReferences:
        - "*"
        repository: "git.local/gitea/signatures"
        attestors:
        - count: 1
          entries:
          - keys:
              publicKeys: |-
                -----BEGIN PUBLIC KEY-----
                MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE4HP6Ra2KaCIv4P9uo6eyyNAWwGhv
                3hj80XA+qMlyasOTo/K1deFyzEDOfPQh751I05Wr3Mn4rWyk3aTCYHFpDQ==
                -----END PUBLIC KEY-----
```

### Sertifikater
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: check-image-demo
spec:
  background: true
  rules:
  - match:
      any:
      - resources:
          kinds:
          - Pod
          namespaces:
          - nyan
    name: verify-signature
    verifyImages:
    - attestors:
      - entries:
        - certificates:
            certChain: |-
              -----BEGIN CERTIFICATE-----
              MIIDojCCAoqgAwIBAgIUTQLqzBaqMfEUsBUJnFSCgKy6OO0wDQYJKoZIhvcNAQEL
              BQAwFjEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMjMwNDI2MDU1ODUwWhcNMjgw
              NDI0MDU1OTIwWjAtMSswKQYDVQQDEyJleGFtcGxlLmNvbSBJbnRlcm1lZGlhdGUg
              QXV0aG9yaXR5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAvKxVCdzN
              JRTION/Yw0CtQAGmeo66BNp+L3zJ+qYy/A/lyxYaZSZUzkkv1/miwdxGD+2RgGNr
              K9FJOx6+DwR7rmVU2NJiEgY/nqpk63vab9JBHk4+BhYDlMCVYJKCn6pLZ1GnGZnv
              w05zhO1zk/YuT8YUMj4HT20Mc0V/NSzTppq0DBrAymoFoA2PvmLPhvqM+HqD5xkC
              6TYh5oWCVGMSWSSJcE97bPUkIa2+13ZJIn3vfUOsiZnS2er61PoDUtdx41mccQ3s
              oxKPPF937QELYaqCYhI4OAwvckd4U+Iy6VQ1zKvaIflYCO0d1KRWOms+6eKcx8RG
              Ml4szfua2Nh+7QIDAQABo4HQMIHNMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8E
              BTADAQH/MB0GA1UdDgQWBBTCMBYPHRokTKIkPJ5bQOZ1zRi3EzAfBgNVHSMEGDAW
              gBRgcto9dUQ9wOtk2OBa/Ijut5nYCDA5BggrBgEFBQcBAQQtMCswKQYIKwYBBQUH
              MAKGHWh0dHBzOi8vdmF1bHQubG9jYWwvdjEvcGtpL2NhMC8GA1UdHwQoMCYwJKAi
              oCCGHmh0dHBzOi8vdmF1bHQubG9jYWwvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsF
              AAOCAQEAXKGn5NdCTqfTgW0r45LInU9bOEbSNpUonePSpuy4GVJx+Sf9CIQT6hvB
              IWlcsK8Fi44zXqtsuZxnsi4LHsZJZ/+aE1xwDjNcx7Jpk94qKEaXt4XGztbG/bZJ
              TvQ5iLP8TSx0CvKkVH+jOtKUFwey2utOdWjdNn5YJ5i8mrmhL+gDfYnzYB2BMNVq
              VOiFuZTKswDxzQsWQG9jPeYeUcjitX3FaVTWQPCN7UmsnToy2RuNFSLF/RzUmrY9
              pt7RpmKe3XK+OLB6R1e0RqTnOcmYGc9mZiHvOP/Mv8fhyDX1HpO5WaRHdqKhgBq9
              XPbne+1g+ICyjYzYG9LpTFz4hQ7Q+Q==
              -----END CERTIFICATE-----
      imageReferences:
      - '*'
      mutateDigest: true
      repository: git.local/gitea/signatures
      required: true
      verifyDigest: true
  validationFailureAction: Enforce
```
Signer ved å bruke følgende:
```shell
cosign sign --key import-cosign.key --yes --cert cert.crt --cert-chain cert-chain.crt curlimages/curl
```

Lagre innholdet i egen yaml fil og kjør det opp i `kyverno` namespace.
```shell
kubectl apply -f check-image-policy.yml -n kyverno
```

### Verifiser

Verifiser at det fungerer ved å prøve å kjøre opp en pod som ikke er signert:
```shell
kubectl run curl-pod --rm -it --image=curlimages/curl -n nyan -- sh
```

Signer så `curlimages/curl` og prøv å kjøre samme kommando på nytt.

## Annet

### Environment variabler
```shell
SIGSTORE_ROOT_FILE=./CA.crts
COSIGN_REPOSITORY=git.local/gitea/signatures
```
- `COSIGN_REPOSITORY`: hvis man ønsker et eget repo for signaturene, f.eks hvis man ikke har skrivetilgang til container registerets om er et tilfelle om man ønsker å signere et image fra en tredje part.