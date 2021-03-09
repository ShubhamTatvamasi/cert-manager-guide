# cert-manager-guide

add repo for cert-manager
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

install cert-manager
```bash
helm install cert-manager jetstack/cert-manager \
  --create-namespace \
  --namespace cert-manager \
  --set installCRDs=true
```

Setup ClusterIssuer:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-cluster-issuer
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```
---

Sample Ingress:
```bash
kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
    - hosts:
      - nginx.k8s.shubhamtatvamasi.com
      secretName: letsencrypt-nginx
  rules:
    - host: nginx.k8s.shubhamtatvamasi.com
      http:
        paths:
        - backend:
            serviceName: nginx
            servicePort: 80
EOF
```
---

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


