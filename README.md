# cert-manager-guide

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.19.1/cert-manager.yaml
```

---

add repo for cert-manager
```bash
helm repo add cert-manager https://charts.jetstack.io
helm repo update
```

install cert-manager
```bash
helm upgrade -i cert-manager cert-manager/cert-manager \
  --create-namespace \
  --namespace cert-manager \
  --set crds.enabled=true \
  --set config.featureGates.ExperimentalGatewayAPISupport=true \
  --set config.featureGates.ACMEHTTP01IngressPathTypeExact=false
```

Install cert-manager with GatewayAPI support:
```bash
helm upgrade -i cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --create-namespace \
  --namespace cert-manager \
  --set config.apiVersion="controller.config.cert-manager.io/v1alpha1" \
  --set config.kind="ControllerConfiguration" \
  --set config.enableGatewayAPI=true
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
      name: letsencrypt-issuer
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
EOF
```

Staging:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-staging-issuer
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
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


