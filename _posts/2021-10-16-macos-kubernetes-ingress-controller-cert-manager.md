## Setup Ingress Controller and Cert Manager on Kubernetes on macOS

Enable Kubernetes in Docker Desktop for macOS.

Install Ingress Controller:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm -n ingress-nginx upgrade --install ingress-nginx ingress-nginx/ingress-nginx --version 4.0.6 --create-namespace --reuse-values
```

Default configuration of **ingress-nginx** is ok, no additional configuration is needed.

Prepare to install **cert-manager** ([just in case](https://github.com/jetstack/cert-manager/issues/2602#issuecomment-636369696)):
```
kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook || true
kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io cert-manager-webhook || true
kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.crds.yaml || true
kubectl -n cert-manager delete secret cert-manager-webhook-ca
kubectl delete namespace cert-manager || true
kubectl create namespace cert-manager
```

Install **cert-manager**:
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm -n cert-manager upgrade --install cert-manager jetstack/cert-manager --create-namespace --version v1.5.4 --reuse-values --set installCRDs=true --set prometheus.enabled=false
```

Configuration using CA Issuer:
```bash
docker run --platform linux/amd64 -e domain=localhost --name mkcert -v ~/Documents/root-ca:/root/.local/share/mkcert vishnunair/docker-mkcert

sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/Documents/root-ca/rootCA.pem

kubectl -n cert-manager create secret tls root-ca --cert=/Users/user/Documents/root-ca/rootCA.pem --key=/Users/user/Documents/root-ca/rootCA-key.pem

cat <<EOF | kubectl -n cert-manager apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: ca-cluster-issuer
  namespace: cert-manager
spec:
  ca:
    secretName: root-ca
EOF

kubectl -n cert-manager delete secret cert-manager-webhook-ca
```

Test the configuration:
```bash
kubectl create ingress ingdefault --class=nginx --default-backend=defaultsvc:http --rule="web.127.0.0.1.nip.io/*=kubernetes:443,tls=web.127.0.0.1.nip.io" --annotation="cert-manager.io/cluster-issuer=ca-cluster-issuer" --annotation="nginx.ingress.kubernetes.io/backend-protocol=HTTPS"

curl --verbose https://web.127.0.0.1.nip.io/healthz

kubectl delete certificate web.127.0.0.1.nip.io
kubectl delete secret web.127.0.0.1.nip.io
kubectl delete ingress ingdefault
```
