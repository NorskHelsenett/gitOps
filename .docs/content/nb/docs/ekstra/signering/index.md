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

## Sigstore/Cosign
Produktet vi skal vise frem heter [Cosign fra Sigstore](https://docs.sigstore.dev/) og det er en enkel måte å signere images, artifakter, BOM osv...

### Installasjon
Oppdatert guide er alltid tilgjengelig på [Cosign Installation](https://docs.sigstore.dev/cosign/installation/)

#### GO
```shell
go install github.com/sigstore/cosign/v2/cmd/cosign@latest
```

#### Homebrew
```shell
brew install cosign
```

#### Binary
```shell
ARCH=$(dpkg --print-architecture)
wget "https://github.com/sigstore/cosign/releases/download/v2.0.0/cosign-linux-$ARCH"
sudo mv cosign-linux-$ARCH /usr/local/bin/cosign
sudo chmod +x /usr/local/bin/cosign
```

### Nøkkelpar
Her brukes nøkkelpar (private/public key) for å signere og verifisere images.

### Key Management System (KMS)
Her benytter man en ekstern aktør for å holde på og sikre privatnøkkelen. Et bra KMS vil aldri kunne gi fra seg en privatnøkkel, det eneste man får tilgang på er den offentlige nøkkelen.

All signering skjer i selve KMS. Ved signering av images er det digest verdien som blir sendt til KMS for signering.

### Sertifikater
Sertifikat signering og sertifisering er best brukt i større organisasjoner, der en organisasjon kan utstede ut fra et eget sertifikat, et nytt sertifikat for hvert enkelt team. Ut fra dette `leaf` sertifikatet som inneholder tre komponenter, en privatnøkkel, en offentlig nøkkel for sertifikatet og den offentlige nøkkelen til utsteder, kan man generere en cosign nøkkel for signering. En organisasjon vil så sette krav til at alle signeringer referer til denne tillitskjeden av sertifikater, og at alle har opphav fra organisasjons sertifikatet.

### OIDC
**O**pen **ID** **C**onnect

---

## Nøkkelpar

Signering med nøkkelpar er den enkleste formen for signering, og kan enkelt integreres i ethvert team.

```shell
cosign generate-key-pair
cosign sign --key cosign.key alpine/curl
```

Privat nøkkel må sikres på et forsvarlig sted.

## KMS

I dette eksemplet skal vi bruke [HashiCorp Vault](/vault/) som KMS tilbyder. Dette forutsetter at transit engine er skrudd på ihht dokumentasjonen.

```shell
cosign generate-key-pair --kms hashivault://signature

cosign sign --kms hashivault://signature git.local/jonasbg/nyancat
```

## Sertifikater

For å ta i bruk sertifikater må man lage en sertifikat kjede som er gyldig. Dette kan du gjøre med følgende fremgangsmetode:

1. Generate a private key for Root certificate

```shell
# Husk å bruk passord her, ellers funker det ikke!
$ openssl genrsa -des3 -out rootCA.key 2048
```

2. Generate Root certificate

```shell
$ openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.crt
```

in Certificate generation set following values
```
C = IN, ST = DEL, L = DEL, O = example.com, OU = sigstore, CN = sigstore, emailAddress = foo@example.com
```


3. Generate Private key for Intermediate certificate

```shell
$ openssl genrsa -out intermediateCA.key 2048
```

4. Generate CSR for Intermediate certificate

```shell
$ openssl req -new -key intermediateCA.key -out intermediateCA.csr
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
$ openssl x509 -req -in intermediateCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -CAserial intermediateca.srl -out intermediateCA.crt -days 1825 -sha256 -extfile intermediateConfigFile
```

7. Create Private key for leaf certificate

```shell
$ openssl genrsa -out leafCA.key 2048
```

8. Create CSR for Leaf certificate

```shell
$ openssl req -new -key leafCA.key -out leafCA.csr
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
$ openssl x509 -req -in leafCA.csr -CA intermediateCA.crt -CAkey intermediateCA.key -CAcreateserial -CAserial leafca.srl -out leafCA.crt -days 1825 -sha256 -extfile leafConfigFile
```



11. Generate Certificate chain by concatinating Intermediate certificate and Root certificate

```shell
$ cat intermediateCA.crt rootCA.crt > certChain.crt
```



12. Import key

```shell
cosign import-key-pair --key leafCA.key
```



13. Sign cosign

```shell
# Fungerer bare med verifisering
cosign sign --key import-cosign.key alpine/git

cosign sign --key import-cosign.key --yes --cert leafCA.crt --cert-chain intermediateCA.crt  docker.io/drone/drone:2.12.1

```

14. Verify signature
```shell
./cosign-linux-arm64 verify alpine/git --cert leafCA.crt --cert-chain certChain.crt | jq .

./cosign-linux-arm64 verify git.local/jonasbg/nyancat --cert leafCA.crt --cert-chain intermediateCA.crt | jq
```

15. Check status
```shell
kubectl get AdmissionReport -A
```

#### ENV variabler
```shell
SIGSTORE_ROOT_FILE=./CA.crts
COSIGN_REPOSITORY=git.local/gitea/signatures
```