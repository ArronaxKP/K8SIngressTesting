# KubernetesTesting
Just a personal testing repository

## Install K8s

I used Docker Desktop

## Install Nginx Controller

I used helm as I am lazy

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx
```

## Apply YAML files

```bash
kubectl apply -f .
```

## How to make a pod not ready

I did this by setting the readiness probe to check for the host.html file. If the file vanishes the probe fails:

To debug this I was exec'ing onto the pod and running these commands:

``` bash
# Make pod not ready
mv /usr/local/apache2/htdocs/host.html /usr/local/apache2/htdocs/host.html2
# Make pod ready
mv /usr/local/apache2/htdocs/host.html2 /usr/local/apache2/htdocs/host.html
```

> Outcome Nginx does not work, as soon as the pod is not ready nginx removes it from the pool and does not route any traffic to that pod. Found out that nginx only uses the readiness probe and not the liveliness probe

## Remove Nginx Ingress Controller

helm uninstall nginx-ingress

## Install HAProxy (from HAProxyTeh)

```bash
helm repo add haproxytech https://haproxytech.github.io/helm-charts
helm repo update
helm install haproxy-ingress haproxytech/kubernetes-ingress \
  --set controller.kind=DaemonSet \
  --set controller.ingressClass=haproxy \
  --set controller.service.type=LoadBalancer
```

> Same issue as Nginx = failed

## Trying this HAProxy - https://github.com/jcmoraisjr/haproxy-ingress

First to install it:

```bash
kubectl create -f https://haproxy-ingress.github.io/resources/haproxy-ingress.yaml
kubectl label node docker-desktop role=ingress-controller
```

Check its running:

```bash
kubectl -n ingress-controller get daemonset
kubectl -n ingress-controller get pod
```

Now deploy the load balancer

```bash
kubectl apply -f jcmoraisjr-loadbalancer.yaml
```

Draining is still not working when tested. We need to enable [drain-support flag](https://haproxy-ingress.github.io/docs/configuration/keys/#drain-support)

```bash
kubectl edit configmap haproxy-ingress -n ingress-controller
```

So the YAML looks like this:

```bash
apiVersion: v1
data:
  drain-support: "true"
kind: ConfigMap
metadata:
  creationTimestamp: "2020-09-12T13:57:35Z"
  name: haproxy-ingress
  namespace: ingress-controller
  resourceVersion: "192909"
  selfLink: /api/v1/namespaces/ingress-controller/configmaps/haproxy-ingress
  uid: beca0398-69d6-47eb-b9b7-bd7c318f49a4
```

Restart the ingress pod (not sure its required)

```bash
kubectl delete pod haproxy-ingress-wn9hd -n ingress-controller
```

It works

## Istio

Download and install istioctl- https://istio.io/latest/docs/setup/getting-started/

```bash
curl -L https://istio.io/downloadIstio | sh -
```

Install istio and lavel namespace for auto proxy envoy

```bash
istioctl install --set profile=demo
kubectl label namespace default istio-injection=enabled
```

Deploy deployment and service:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Then apply istio config (virtaul service, gateway and destination rule)

```bash
kubectl apply -f istio-gateway.yaml
```

Test - session stickyness works - limits scaling (up or down) recreates all cookies and re-routes all traffic.

There is no way to drain sessions using this ingress.
