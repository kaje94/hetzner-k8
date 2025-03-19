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
k3s kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
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

## Setup Helm

> Needed for Prometheus & Grafana

### Install [Helm](https://helm.sh/docs/intro/install/)

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

Following is needed if helm fails to find k3s cluster and throws a cluster unreachable error

```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Setup Prometheus & Grafana

Follow instructions in https://medium.com/@gayatripawar401/deploy-prometheus-and-grafana-on-kubernetes-using-helm-5aa9d4fbae66 to install Prometheus & Grafana

### Setup Prometheus

To add Prometheus to helm repo

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

To install Prometheus

```
helm install prometheus prometheus-community/prometheus
```

To Make Prometheus accessible via NodePort 30003

```
k3s kubectl patch service prometheus-server --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"},{"op":"replace","path":"/spec/ports/0/nodePort","value":30003}]'
```

### Setup Grafana

To add Grafana to helm repo

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

To install Grafana

```
helm install grafana grafana/grafana
```

To Make Grafana accessible via NodePort 30004

```
k3s kubectl patch service grafana --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"},{"op":"replace","path":"/spec/ports/0/nodePort","value":30004}]'
```

To get grafana admin user password

```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Add Premetheus as data source for Grafana

1. Login to grafana via http://[ip]:30004, with username admin and password from previous step
2. Click connections > data source > Add new Data source
3. Select grafana and provide http://[ip]:30003 as prometheus server url
4. Follow steps in the guide to add k8 dashboard to Grafana

## Adding Persistent Volume and Persistent Volume claim

To retain data across pod restarts or re-deployments, you need to use Persistent Volumes (PVs) and Persistent Volume Claims (PVCs). Otherwise, their data will be lost when the pods are restarted or rescheduled.

> Make sure to update the node names in /pv-config yamls

> Refer guide

Create relevant directories

```
sudo mkdir -p /grafana-data
sudo chown -R 472:472 /grafana-data  # Adjust for Grafana UID
```

```
sudo mkdir -p /prometheus-data
sudo chown -R 65534:65534 /prometheus-data
```

Create PV and PVC for both Prometheus and Grafana

```
k3s kubectl apply -f ./pv-configs
```
