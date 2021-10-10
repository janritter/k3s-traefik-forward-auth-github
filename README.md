# K3S - Traefik forward auth with GitHub

Kubernetes resources to setup [traefik forward auth](https://github.com/thomseddon/traefik-forward-auth) with GitHub as OAuth provider

This was tested with K3S, but should also work in other K8S installations.

## Setup

### Variables which must be adapted

- Environment variables in `deployment.yaml`
  - `PROVIDERS_GENERIC_OAUTH_CLIENT_ID`
  - `COOKIE_DOMAIN`
  - `AUTH_HOST`
- In `ingress.yaml`
  - Hosts (`auth.example.com`)
  - Also a certificate source must be configured (e.g. cert-manager annotation)
- Secrets in `secret.yaml`
  - Both secret values must be base64 encoded (see instructions below)

### Base64 encoding secrets

```bash
echo -n 'secret-value' | base64
```

### Configuring GitHub OAuth App

To get the `PROVIDERS_GENERIC_OAUTH_CLIENT_ID` and `PROVIDERS_GENERIC_OAUTH_CLIENT_SECRET` values for GitHub, you have to configure a GitHub OAuth App.

#### Values for the GitHub App

- Homepage URL: https://example.com
- Callback URL: https://auth.example.com/_oauth

## Usage

To enable traefik-forward-auth in your ingress, add the following annotation:
```yaml
traefik.ingress.kubernetes.io/router.middlewares: traefik-auth-forward-auth@kubernetescrd
```

### Full ingress example with annotation

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-service-ingress
  namespace: test
  annotations:
    kubernetes.io/ingress.class: traefik
    cert-manager.io/cluster-issuer: lets-encrypt
    traefik.ingress.kubernetes.io/router.middlewares: traefik-auth-forward-auth@kubernetescrd
spec:
  tls:
   - secretName: test-service-tls
     hosts:
      - test-service.example.com
  rules:
   - host: test-service.example.com
     http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: test-service-service
              port:
                number: 80
```
