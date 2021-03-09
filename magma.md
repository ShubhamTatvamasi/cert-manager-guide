# Magma cert-manager

Create a self signed issuer:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
EOF
```

Create self signed certificate:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: root-ca
spec:
  secretName: root-ca
  dnsNames:
  - "*.magma.shubhamtatvamasi.com"
  issuerRef:
    name: selfsigned-issuer
EOF
```

Create root CA certificate:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: magma-root-ca
spec:
  secretName: magma-root-ca-tls
  commonName: magma.svc.cluster.local
  usages:
    - server auth
    - client auth
  isCA: true
  issuerRef:
    name: selfsigned-issuer
EOF
```


Create CA issuer:
```bash
kubectl apply -f - << EOF
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: ca-issuer
spec:
  ca:
    secretName: magma-root-ca-tls
EOF
```

Check root certificate details:
```bash
kubectl get secrets magma-root-ca-tls \
  -o jsonpath='{.data.tls\.crt}' | \
  base64 -d | openssl x509 -text -noout -in -
```


