---
title: "Minikube w/ cert manager"
date: 2024-04-13T17:35:00-04:00
draft: false
---
### Using cert manager with Minikube
<br><hr>
1. Create your Root CA and key
   ```shell
   openssl genrsa -out rootCAKey-kong.pem 2048
   openssl req -x509 -sha256 -new -nodes -key rootCAKey-kong.pem -days 3650 -out rootCACert-kong.pem
   ```

2. Import the CA into your browser or OS trust store
3. Install Cert Manager
   ```shell
   kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
   ```
4. Create a secret in the cert-manager namespace to hold your root CA
   ```shell
   kubectl create secret tls rootca --key="rootCAKey-kong.pem" --cert="rootCACert-kong.pem" -n cert-manager
   ```
5. Create the Cert Manager ClusterIssuer
   ```shell
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: kongfused-issuer
    spec:
      ca:
        secretName: rootca
   ```
6. Create the Cert Manager Certificate. Update your DNS names as needed
   ```shell
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: kongfused-proxy
    spec:
      secretName: kongfused-cert-tls
      duration: 240h # 10d
      renewBefore: 48h # 2d
      subject:
        organizations:
        - kongfused.com
      isCA: false
      commonName: kongfused.com
      dnsNames:
        - "kongfused.com"
        - "admin.kongfused.com"
      privateKey:
        rotationPolicy: Always
        algorithm: ECDSA
        encoding: PKCS8
        size: 256
      usages:
        - server auth
      issuerRef:
        name: kongfused-issuer
        kind: ClusterIssuer
   ```