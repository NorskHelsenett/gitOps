---
title: 'Multipass'
date: 2019-02-11T19:27:37+10:00
weight: 2
---

For å kunne starte, må du ha tilgang til et kubernetes kluster. Et av de enklere verktøy for å få til dette er multipass som oppretter en VM der man installerer microk8s. Det finnes mange gode alternativer som er mye mer lettvektere, men dette er valgt fordi multipass kan brukes til andre formål, og microk8s er et godt alternativ for å kjøre kubernetes på low-end maskinvare som Raspberry Pi.

For å installere multipass følg den offisielle installasjons guiden på [installasjons guide](https://multipass.run/install)

### Alternativer til multipass og microk8s

- [minikube](https://minikube.sigs.k8s.io/docs/) kanskje det enkleste verktøyet for å kjøre opp et kubernetes miljø? Med mulighet for forskjellig vm tilbyder.
- [k3s](https://k3s.io) produksjonsklart kubernetes med kun de nødvendigste kubernetes funksjonene. Minimalt og god på ytelse.
- [k3d](https://k3d.io/v5.4.9/) **k3**s in **d**ocker. Nedstrippet versjon av k3s som kjører i docker.
- [kind](https://kind.sigs.k8s.io) fullverdig verktøy for å kjøre opp kubernetes i docker.
- [k0s](https://k0sproject.io) minimalt kubernetes system som er enkelt å installere og provsjonere.

### Installer microk8s

```shell
multipass launch --name microk8s-vm --memory 4G --disk 40G
multipass shell microk8s-vm
sudo snap install microk8s --classic --channel=1.26/stable
sudo iptables -P FORWARD ACCEPT

sudo usermod -a -G microk8s $USER
mkdir ~/.kube
sudo chown -f -R $USER ~/.kube
newgrp microk8s

microk8s enable ingress dns cert-manager hostpath-storage
```

For å få argocd ingressen til å fungere må nginx kjøres med `--enable-ssl-passthrough`. Dette patches ved følgende kommando.

```shell
kubectl -n ingress patch daemonsets nginx-ingress-microk8s-controller \
--type=json \
-p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-ssl-passthrough"}]'
```

Kjør følgende kommando og legg resultatet i `hosts` filen på egen maskin og multipass. På MacOSX og Linux er den rette plasseringen `/etc/hosts`.

**Det er viktig at du gjør dette på multipass VM instansen din. Og dessverre vil denne filen resettes etter hver restart av maskinen**
```shell
IP=$(hostname -I | awk '{print $1}')
cat << EOF
  $IP git.local
  $IP drone.local
  $IP nyan.local
  $IP argocd.local
  $IP vault.local
EOF
```

### Kommandoer du bør vite om

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


Sjekk noder og pod tilstander
```shell
kubectl get nodes
kubectl get pods -A
```

Opprett shell til vm
```shell
multipass shell microk8s-vm
```

Stopp Vm
```shell
multipass stop microk8s-vm
```

Slett VM og rydd opp
```shell
multipass delete microk8s-vm
multipass purge
```