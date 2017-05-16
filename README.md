# k8s-federation-azure
Kubernetes cluster federation on Azure with CoreDNS

## Prerequisites
Kubernetes 1.6+ with Beta APIs enabled

## Deploying CoreDNS and etcd charts

`helm install --namespace my-namespace --name etcd-operator stable/etcd-operator`

`helm upgrade --namespace my-namespace --set cluster.enabled=true etcd-operator stable/etcd-operator`

values.yaml
```
isClusterService: false
serviceType: "LoadBalancer"
middleware:
  kubernetes:
    enabled: false
  etcd:
    enabled: true
    zones:
    - "example.com."
    endpoint: "http://etcd-cluster.etcd-operator:2379"
```

`helm install --namespace etcd-operator --name coredns -f Values.yaml stable/coredns`

## Provision federated control plane

coredns-provider.conf
```
[Global]
etcd-endpoints = http://etcd-cluster.etcd-operator:2379
zones = azure.sertac.club.
```

```
kubefed init federation \
    --host-cluster-context=eastus \
    --dns-provider=coredns \
    --dns-provider-config=coredns-provider.conf
```   

## Joining clusters to federation

`kubectl config use-context federation`

`kubefed join eastus --host-cluster-context=eastus`

`kubefed join westus --host-cluster-context=eastus`

## Create federated replicaset

`kubectl create -f https://github.com/kelseyhightower/kubernetes-cluster-federation/blob/master/rs/nginx.yaml`

## Create federated service

`kubectl create -f https://github.com/kelseyhightower/kubernetes-cluster-federation/blob/master/services/nginx.yaml`

## Add Azure DNS Zone
