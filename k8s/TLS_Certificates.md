# TLS Certificates

We use `cert-manager`, CloudFlare, and Let's Encrypt.

Install `cert-manager`.

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.3/cert-manager.yaml

Create a CloudFlare API token in the Web UI. Store it as a secret in the `cert-manager` namespace.

    apiVersion: v1
    kind: Secret
    metadata:
      namespace: cert-manager
      name: cloudflare-api-token
    type: Opaque
    stringData:
      api-token: "0123abcd"

Create an issuer for Let's Encrypt in the `cert-manager` namespace.

    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      namespace: cert-manager
      name: letsencrypt-issuer
    spec:
      acme:
        server: https://acme-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
          name: letsencrypt-account-key
        solvers:
        - dns01:
            cloudflare:
              apiTokenSecretRef:
                name: cloudflare-api-token
                key: api-token

Create certificates in the namespaces where they will be used.

    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: instantramen-tls
    spec:
      secretName: instantramen-tls
      issuerRef:
        kind: ClusterIssuer
        name: letsencrypt-issuer
      privateKey:
        algorithm: RSA
        encoding: PKCS1
        size: 2048
      duration: 2160h # 90d
      renewBefore: 360h # 15d
      dnsNames:
        - "instantra.men"
        - "*.instantra.men"
