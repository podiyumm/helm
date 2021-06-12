# helm

## Local Testing

Run k8s cluster locally using k3d:
```shell
 k3d cluster create podiyumm -p "8123:30000@agent[0]" --agents 1
```

Deploy Tyk GW:
```shell
#helm repo add bitnami https://charts.bitnami.com/bitnami
#helm repo update
kubectl create namespace tyk
#helm install tyk-redis bitnami/redis -n tyk
kubectl apply -f deploy/dependencies/redis.yaml -n tyk
helm upgrade --install -f ./tyk-headless/values.yaml tyk-headless ./tyk-headless  -n tyk
```

Test Tyk GW:
```shell
http :8123/ 
```

