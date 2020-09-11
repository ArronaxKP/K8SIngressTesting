# KubernetesTesting
Just a personal testing repository


## Install K8s

I used Docker Desktop

## Install Nginx Controller

I used helm as I am lazy

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-release ingress-nginx/ingress-nginx
```

## Apply YAML files

```bash
kubectl apply -f .
```

## Outcome Nginx does not work
