---
title: "Multipass"
description: ""
lead: ""
date: 2023-04-18T09:51:05Z
lastmod: 2023-04-18T09:51:05Z
draft: false
images: []
menu:
  docs:
    parent: "gitops"
    identifier: "multipass-18906d7db78e8b356435a61bbfe3b2d9"
weight: 200
toc: true
type: docs
---

For å kunne starte, må du ha tilgang til et kubernetes kluster. Et av de enklere verktøy for å få til dette er multipass som oppretter en VM der man installerer microk8s. Det finnes mange gode alternativer som er mye mer lettvektere, men dette er valgt fordi multipass kan brukes til andre formål, og microk8s er et godt alternativ for å kjøre kubernetes på low-end maskinvare som Raspberry Pi.

For å installere multipass følg den offisielle installasjons guiden på [installasjons guide](https://multipass.run/install)

## Alternativer til multipass og microk8s

- [minikube](https://minikube.sigs.k8s.io/docs/) kanskje det enkleste verktøyet for å kjøre opp et kubernetes miljø? Med mulighet for forskjellig vm tilbydere.
- [k3s](https://k3s.io) produksjonsklart kubernetes med kun de nødvendigste kubernetes funksjonene. Minimalt og god på ytelse.
- [k3d](https://k3d.io/v5.4.9/) **k3**s in **d**ocker. Nedstrippet versjon av k3s som kjører i docker.
- [kind](https://kind.sigs.k8s.io) fullverdig verktøy for å kjøre opp kubernetes i docker.
- [k0s](https://k0sproject.io) minimalt kubernetes system som er enkelt å installere og provsjonere.

## Provisjon ny virtuell maskin med Multipass
```shell
multipass launch --name microk8s-vm --memory 8G --disk 40G --cpu 4
multipass shell microk8s-vm
```

## Installer microk8s
### Linux

```shell
# installer microk8s
sudo snap install microk8s --classic --channel=1.27/stable

# legg bruker til microk8s gruppen
sudo usermod -a -G microk8s $USER
mkdir ~/.kube
sudo chown -f -R $USER ~/.kube
newgrp microk8s

# Test at det fungerer
microk8s status

# Installer addons
microk8s enable ingress dns cert-manager hostpath-storage host-access
```

- **ingress:** Dette oppretter en ingress (reverse-proxy) som videresender trafikk for et FQDN (example.com) til tilhørende service som så svarer på responsen.
- **dns:** Dette oppretter en DNS tilbyder i klusteret så vi kan kalle på tjenestene ved å bruke FQDN til klusteret (example.svc.cluster.local)
- **cert-manager:** Cert-manager er en tilbyder for å utstede sertifikater, enten selvsignerte eller gjennom eksterne tjenester som Let's Encrypt
- **hostpath-storage:** Dette er en enkel tilbyder for å tilby lagring inn i noden ved å lagre data på selve ubuntu hosten.
- **host-access:** Dette tilgjengeliggjør host maskinenens ressurser inn i klusteret. Vi bruker dette kun for å forenkle bruken av DNS oppslag mot [drone.local](https://drone.local) og [git.local](https://git.local) inne i klusteret.

{{< alert icon="🚨" context="warning" text="Ikke bruk host-access i vanlige klustre uten å ha gjort en skikkelig vurdering! Det gjøres kun her for å forenkle DNS oppslag mot interne ressurser som https://git.local" />}}

### Windows
For å få microk8s til å fungere på Windows ved å bruke WSL må man aktivere versjon 2 av WSL og skru på `systemd`.

1. Oppdaterer WSL til å kjøre på versjon 2
```powershell
wsl --update
wsl --set-default-version 2
```
2. Etter installering av ubuntu, logg inn i WSL og aktiver `systemd`
```shell
echo -e "[boot]\nsystemd=true" | sudo tee /etc/wsl.conf
```
3. Logg ut av ubuntu og restart WSL fra powershell
```powershell
wsl --shutdown
wsl
```
4. Følg veiledning for [Linux](#linux)

{{< alert icon="🪟" context="info" text="På windows så finner du hosts filen på følgende filsti, c:\Windows\System32\drivers\etc\hosts" />}}

### hosts

Kjør følgende kommando og legg resultatet i `hosts` filen på egen maskin og multipass. På MacOSX og Linux er den rette plasseringen `/etc/hosts`.

**Det er viktig at du gjør dette på multipass VM instansen din. Og dessverre vil denne filen resettes etter hver restart av maskinen**
```shell
IP=$(hostname -I | awk '{print $1}' )
cat << EOF | sudo tee -a /etc/hosts
  $IP git.local
  $IP drone.local
  $IP nyan.local
  $IP argocd.local
  $IP vault.local
EOF
```

## TIPS

### Microk8s

#### Status
```shell
microk8s status
```

#### Config
Konfigfilen kan eksponeres til fil på følgende måte, slik at man kan bruke egen kubectl, eller andre verktøy som [k9s](https://k9scli.io/).
```shell
microk8s config > ~/.kube/config
```

### Kubectl

#### Alias
Alias for enklere kommandoer
```shell
alias kubectl="microk8s kubectl"
alias k=kubectl
```
For å persistere kommandoene over, må de legges inn i .-rc fil som `.bashrc` eller `.zshrc`.
```shell
cat << EOF >> ~/.bashrc
  alias kubectl="microk8s kubectl"
  alias k=kubectl
EOF
source ~/.bashrc
```
#### Interaksjon med klusteret
Sjekk noder og pod tilstander
```shell
kubectl get nodes
kubectl get pods -A
```

#### Completion
Auto completion gir deg automatisk utfylling ved bruk av [tab] slik at det er enkelt å gjøre handlinger mot klusteret. [Ler mer](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

```shell
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

alias k=kubectl
complete -o default -F __start_kubectl k
```

#### Smarte kubernetes kommandoer

Liste ut alle oversikter over ressurser i et kluster

```shell
kubectl api-resources
```

List ut spesifikasjonen for ressurser i klusteret
```shell
kubectl explain application.spec
```

### k9s

[k9s](https://k9scli.io) er et visuelt CLI verktøy for å lett få en oversikt over ressurser i klusteret, blant annet loggin, shell, metrikler osv.

```shell
ARCH=$(dpkg --print-architecture)
wget https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_$ARCH.tar.gz
tar xvf k9s_Linux_$ARCH.tar.gz
sudo mv k9s /usr/local/bin/k9s
rm README.md LICENSE k9s_Linux_$ARCH.tar.gz
```

Husk å hente ut kubernetes config for at k9s skal fungere, [her er veiledning](#config)

### jq
Installere jq
```shell
sudo apt install jq -y
```
### Multipass

#### Shell
Opprett shell til VM
```shell
multipass shell microk8s-vm
```
#### Stopping
Stopp VM
```shell
multipass stop microk8s-vm
```
#### Sletting
Slett VM og rydd opp
```shell
multipass delete microk8s-vm
multipass purge
```