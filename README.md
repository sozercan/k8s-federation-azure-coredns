# k8s-federation-azure
Kubernetes cluster federation on Azure with CoreDNS

`helm install --namespace my-namespace --name etcd-operator stable/etcd-operator`
`helm upgrade --namespace my-namespace --set cluster.enabled=true etcd-operator stable/etcd-operator`

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
    endpoint: "http://etcd-cluster.my-namespace:2379"
```

`helm install --namespace my-namespace --name coredns -f Values.yaml stable/coredns`


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

`kubectl config use-context federation`

`kubefed join eastus --host-cluster-context=eastus`
`kubefed join westus --host-cluster-context=eastus`
