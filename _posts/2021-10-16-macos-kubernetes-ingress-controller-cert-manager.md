## Setup Ingress Controller and Cert Manager on Kubernetes on macOS

Enable Kubernetes in Docker Desktop for macOS.

Install Ingress Controller:
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm -n ingress-nginx upgrade --install ingress-nginx ingress-nginx/ingress-nginx --version 4.0.6 --create-namespace --reuse-values
```

Default configuration is ok, no additional configuration is needed.

Install Cert Manager:
```
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm -n cert-manager upgrade --install cert-manager jetstack/cert-manager --create-namespace --version v1.5.4 --reuse-values --set installCRDs=true --set prometheus.enabled=false
```
