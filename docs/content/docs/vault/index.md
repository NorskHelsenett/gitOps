---
title: 'HashiCorp Vault'
date: 2019-02-11T19:27:37+10:00
weight: 8
---

![Vault](https://www.datocms-assets.com/2885/1542059843-vaultshare-imglogo-w-stack-graphic1200x630.png)

HashiCorp Vault er en verktøykasse med sikkerhetsverktøy med blant annet muligheter for å håndtere passord, hemmeligheter, autentisere, autorisere, kryptere, signere osv.

Vi skal bruke Vault til å signere våre images.

## Initialize
Etter installasjon så må vi initalisere vault med å definere antall nøkler, og hvor mange av de som må brukes for å låse opp velvet. Disse nøklene blir lagret på egen maskin.
```shell
kubectl -n vault exec -it vault-0 -- vault operator init -key-shares=5 -key-threshold=2 --format json > vault-secrets.json
```

## Environment variables
Dette er variabler som brukes for å aksessere og interagere med vault.

```shell
export VAULT_TOKEN="$(jq -r .root_token ./vault-secrets.json)"
export VAULT_ADDR=https://vault.local #Kun nødvendig om man bruker vault-cli eller cosign for signering ved bruk av Vault.
```

## Unseal
Når velvet er initialisert, så må vi låse det opp. Disse tre linjene låser det opp for oss ved å velge nøkkel nr 0,2 og 4.

```shell
kubectl exec -n vault -it vault-0 -- vault operator unseal $( jq -r '.unseal_keys_b64[0]' vault-secrets.json)
kubectl exec -n vault -it vault-0 -- vault operator unseal $( jq -r '.unseal_keys_b64[2]' vault-secrets.json)
kubectl exec -n vault -it vault-0 -- vault operator unseal $( jq -r '.unseal_keys_b64[4]' vault-secrets.json)
```

## Tilgang
Vault er nå låst opp og tilgjengelig på https://vault.local, bruk verdien `root_token` fra `vault-secrets.json` for å logge inn. Root token skal kun benyttes for å starte opp vaultet, hvorpå det bør kastes.

> It is generally considered a best practice not to persist root tokens. Use the root token only for just enough initial setup. Once you enabled an auth method with appropriate policies allowing Vault admins to log in and perform operational tasks, the admins should authenticate with Vault instead of using the root token.

For å lage nytt root_token fra unsea_tokens, [les her](https://developer.hashicorp.com/vault/tutorials/operations/generate-root#generate-root)

![Vault](vault.png)

## Transit Engine og signering

For å signere ved bruk av Vault må vi skru på Transit engine, dette gjøres ved å kjøre følgende kommando:

```shell
kubectl exec -n vault -it vault-0 -- sh -c "VAULT_TOKEN=$VAULT_TOKEN vault secrets enable transit"
```