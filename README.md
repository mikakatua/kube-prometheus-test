# Testing the deployment on Kubernetes
Create a namespace for the deployment
```
kubectl create ns monitoring
```

## Minikube
We have to tweak a bit the configuration to avoid the failure of some monitored targets:
```
kubectl create secret generic etcd-certs -n monitoring \
  --from-literal=ca.crt="$(minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/ca.crt)" \
  --from-literal=server.crt="$(minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.crt)" \
  --from-literal=server.key="$(minikube ssh -- sudo cat /var/lib/minikube/certs/etcd/server.crt)"

minikube ssh -- sudo sed -i 's/bind-address=127.0.0.1/bind-address=0.0.0.0/' \
  /etc/kubernetes/manifests/kube-controller-manager.yaml \
  /etc/kubernetes/manifests/kube-scheduler.yaml

minikube ssh -- sudo systemctl restart kubelet

```

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kps prometheus-community/kube-prometheus-stack -n monitoring -f values_minikube.yaml
```