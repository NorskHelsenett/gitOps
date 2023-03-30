# gitOps-intro

Opprett en linux VM med minkube eller annen ønsket k8s tilbyder.
```bash
# Start multipass VM
multipass launch --name demo --cpus 2 --mem 4G --disk 50G minikube
```

## Klassisk push CI/CD
Installer NGINX i klusteret gjennom tradisjonell push metodikk
```bash
kubectl apply -f https://raw.githubusercontent.com/jonasbg/gitOps-intro/main/applications/nginx/nginx-pod.yml
```

### Åpne port-forward for å se resultatet
```bash
kubectl port-forward pod/my-nginx 8080:80 --address 0.0.0.0
```
Nå du nå åpner nettsiden ved å gå til IP for multipass og portnummer 8080 vil du se et "hello world" fra nginx.

# Fortsett
For å fortsette går du nå til branchen som heter `gitea` for å installere ArgoCD og Gitea gjennom gitOps.
> 👉 [GITEA](https://github.com/jonasbg/gitOps-intro/tree/gitea)
