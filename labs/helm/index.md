# Working with Helm

## Install Helm 

```
wget https://raw.githubusercontent.com/helm/helm/master/scripts/get -O - | bash
```

Now that Helm is installed we need to install the backend 

Create tiller service account
```
kubectl create serviceaccount tiller --namespace kube-system
```

Grant tiller cluster admin role 
```
kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
```

Initialize Helm to install tiller in your cluster 
```
helm init --service-account=tiller
helm repo update
```

## The Helm Chart File System

The Helm chart's file system. Helm relies upon the existence file, `Chart.yaml` and
a `templates` directory the contains the files:

* `deployment.yaml`
* `service.yaml`
* `ingress.yaml`



```bash
chart_01
├── Chart.yaml
└── templates
    ├── deployment.yaml
    ├── ingress.yaml
    └── service.yaml

```

## Chart Operations for a Simple Chart

**Step 1:** Execute a release

`helm install chart_01 --name=simplechart`

**Step 2:** List the release

`helm list`

**Step 3:** Try the release out. Find the IP address of the service

`kubectl get svc`

Load the 'External-IP' in a browser

**Step 4:** Delete a release

`helm delete simplechart --purge`



## Chart Operations for a Chart Using Value Variables

The Helm chart's file system for Stooges

```bash
chart_stooges/
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── values.yaml
```

Contents of the file, `values.yaml`

```yaml
replicas: 2
color: blue
author: me
```
Instructor will show demo of Chart install with variables. 

