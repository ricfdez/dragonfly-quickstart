# Dragonfly

## Introduction

Dragonfly is a Kubernetes-native Peer-to-Peer (P2P) content delivery system. It allows optimization of container images within kubernetes clusters which reduces the load on external repos and enhances deployment efficiency. In the end, we increase the reliability of the pull operation and reduce network congestion.

## QuickStart

### Requirements:
- helm
- kind
- kubectl

### Steps

1. Stage a [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) cluster yaml with 2 workers

``` bash
## creates kind config, a cluster and changes context
POCD7Y=poc-dragonfly # name cluster to desired value
cat > kind-config.yaml << EOF ; kind create cluster --name $POCD7Y --config kind-config.yaml ; kubectl config use-context kind-$POCD7Y
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF

```
``` connecting to kind cluster - if your terminal dies, or exits, you can configure your kube-config with the below
kind get kubeconfig --name $POCD7Y > kind-kubeconfig.yaml ; export KUBECONFIG=./kind-kubeconfig.yaml

```

2. Pull dragonfly images and loads them in the kind cluster
``` bash 
docker pull dragonflyoss/scheduler:latest
docker pull dragonflyoss/manager:latest
docker pull dragonflyoss/dfdaemon:latest
kind load docker-image dragonflyoss/scheduler:latest --name $POCD7Y
kind load docker-image dragonflyoss/manager:latest --name $POCD7Y
kind load docker-image dragonflyoss/dfdaemon:latest --name $POCD7Y

```

3. Create Dragonfly cluster based of helm charts, and install cluster. You should see the pods in the ```dragonfly-system```  ns if successful

```  bash 
cat > kind-config-chart.yaml << EOF ; helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/ ; helm install --wait --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly -f kind-config-chart.yaml && kubectl get po -n dragonfly-system
containerRuntime:
  containerd:
    enable: true
    injectConfigPath: true
    registries:
      - 'https://ghcr.io'

scheduler:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

seedPeer:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

dfdaemon:
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

manager:
  replicas: 1
  metrics:
    enable: true
  config:
    verbose: true
    pprofPort: 18066

jaeger:
  enable: true
EOF

```
Cleanup

``` bash 
kind delete cluster --name $POCD7Y 

```
Dragonfly is set up at this point.

### Testing

Pulling an image back to source using Dragonfly 
``` docker exec -i kind-$POCD7Y /usr/local/bin/crictl pull ghcr.io/dragonflyoss/dragonfly2/scheduler:v2.0.5 ```

### References
- [Website](https://d7y.io/docs/)
- [Architecture](https://d7y.io/docs/concepts/terminology/architecture/)
- [QuickStart](https://d7y.io/docs/getting-started/quick-start/kubernetes/)

### Glosary
**Preheat:** Process of proactively downloading and caching container images onto nodes within a Kubernetes cluster before they are actually needed for running pods or applications. This anticipatory approach ensures that the required images are readily available on nodes, reducing the latency and time required for pods to start.
