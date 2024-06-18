# Setup Hetzner Server

Following instructions explains how to setup a simple single node Kubernates cluster for testing purposes in Hetzner.

These instructions are specific to K3S and Argo CD.

### Install [K3S](https://k3s.io)

```shell
curl -sfL https://get.k3s.io | sh -
```

### Install [Argo CD](https://argo-cd.readthedocs.io/en/stable/)

Create namespace for Argo CD

```shell
k3s kubectl create namespace argocd
k3s kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Access Argo CD UI

Make Argo CD UI accessible via NodePort 30001

```
k3s kubectl patch service argocd-server -n argocd --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"},{"op":"replace","path":"/spec/ports/0/nodePort","value":30001}]'
```

To set back the service as ClusterIP **(if needed)**

```
k3s kubectl patch service argocd-server -n argocd --type='json' -p '[{"op":"replace","path":"/spec/type","value":"ClusterIP"}]'
```

To get initial Argo CD password for user admin

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

### Argo CD CLI

To install Argo CD CLI

```
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-arm64
chmod +x /usr/local/bin/argocd
```

Login to Argo CD CLI

```
argocd login <Public IP of server>:30001
```

To list the argo servers

```
argocd cluster list
```

### Initialize cluster through Argo CD UI

> TODO

### Setup ingress

```
k3s kubectl apply -f ingress.yaml
```
