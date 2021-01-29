# cert-manager-guide

create namespace
```bash
kubectl create namespace cert-manager
```

add repo for cert-manager
```bash
helm repo add jetstack https://charts.jetstack.io
```

update repo
```bash
helm repo update
```

install cert-manager
```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true
```

test resources
```yaml
kubectl apply -f - << EOF
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
```


