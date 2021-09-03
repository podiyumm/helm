# helm

## Local Testing

### Create k3d cluster
Inspiration: https://k3d.io/usage/guides/exposing_services/

Run k8s cluster locally using k3d:
```shell
k3d cluster create podiyumm --api-port 6550 -p "8122:80@loadbalancer" -p "8123:443@loadbalancer" --agents 2 --k3s-server-arg '--no-deploy=traefik'
export KUBECONFIG="$(k3d kubeconfig write podiyumm)"
```

### Install emissary-gateway

Inspiration: https://www.getambassador.io/docs/emissary/latest/tutorials/getting-started/

```shell
# Add the Repo:
helm repo add datawire https://www.getambassador.io
 
# Create Namespace and Install:
kubectl create namespace ambassador && \
helm install ambassador --namespace ambassador datawire/ambassador && \
kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes
```

check
```shell
open https://localhost:8123
```

### install argocd
Inspiration: https://www.getambassador.io/docs/argo/latest/quick-start/

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
#kubectl create namespace argo-rollouts
#kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```

access argocd
```shell
kubectl port-forward svc/argocd-server -n argocd 8124:443 &
# argo pwd
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

open http://localhost:8124

argocd login localhost:8124
```

### Deploy apps

Deploy apps:
```shell
helm dependencies update ./podiyumm-song
helm upgrade --install -f ./podiyumm-song/values.yaml podiyumm-song ./podiyumm-song

# to check it works
# curl --insecure https://localhost:8123/song/song

helm upgrade --install -f ./podiyumm-musician-ui/values.yaml podiyumm-musician-ui ./podiyumm-musician-ui
```




## All in one
```shell
#kubectl create namespace argocd
#kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
#kubectl port-forward svc/argocd-server -n argocd 8124:443 &
## argo pwd
#export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)


k3d registry create registry.localhost --port 5000

cat << EOF > registries.yaml
mirrors:
  "localhost:5000":
    endpoint:
      - http://k3d-registry.localhost:5000
EOF

k3d cluster delete podiyumm
#  --agents 2
k3d cluster create podiyumm --api-port 6550 -p "8122:80@loadbalancer" -p "8123:443@loadbalancer" --k3s-server-arg '--no-deploy=traefik' 
#--registry-use k3d-registry.localhost:5000 --registry-config registries.yaml
    export KUBECONFIG="$(k3d kubeconfig write podiyumm)"
helm repo add datawire https://www.getambassador.io
kubectl create namespace ambassador && \
helm install ambassador --namespace ambassador datawire/ambassador && \
kubectl -n ambassador wait --for condition=available --timeout=90s deploy -lproduct=aes

helm dependencies update ./podiyumm-song
helm upgrade --install -f ./podiyumm-song/values.yaml podiyumm-song ./podiyumm-song
helm upgrade --install -f ./podiyumm-musician-ui/values.yaml podiyumm-musician-ui ./podiyumm-musician-ui
```


local registry in k3d:
```shell
# https://github.com/rancher/k3d/issues/19

k3d registry create registry.localhost --port 5000

cat << EOF > registries.yaml
mirrors:
  "localhost:5000":
    endpoint:
      - http://k3d-registry.localhost:5000
EOF

k3d cluster create mycluster -p "8081:80@loadbalancer" --registry-use k3d-registry.localhost:5000 --registry-config registries.yaml

# docker push localhost:5000/podiyumm-ws:0.1.0
# and then use it 
# kubectl run myimage --image localhost:5000/myimage:latest


```


using kind instead of (instable) k3d:

```shell
cd kind
./kind-with-registry.sh
```