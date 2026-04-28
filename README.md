# Mailcatcher Helm Chart
This Helm chart installs the Mailcatcher service into a Kubernetes cluster. Mailcatcher is a simple SMTP server and web interface designed for testing email-sending applications by capturing and displaying emails locally instead of sending them to their intended recipients.

## Application Source:

https://mailcatcher.me/

https://github.com/sj26/mailcatcher
## Features
Deploys the Mailcatcher service to your Kubernetes cluster.
Configurable SMTP and HTTP ports.
Lightweight and easy to set up.
Ideal for testing and development environments.
## Prerequisites
Helm 3.0+

A running Kubernetes cluster with sufficient resources.

For Gateway API support: a Gateway API implementation (e.g., [Envoy Gateway](https://gateway.envoyproxy.io/)) installed in the cluster with a `Gateway` resource already provisioned.
## Installation

### Install the Chart
```
helm install mailcatcher . --namespace <your-namespace>
Replace <your-namespace> with the namespace where you want to deploy Mailcatcher.
```

### Configuration
The following table lists the configurable parameters of the Mailcatcher chart and their default values:

#### General

| Parameter                 | Description                              | Default                    |
|---------------------------|------------------------------------------|----------------------------|
| replicaCount              | Number of replicas for the deployment    | 1                          |
| image.repository          | GitHub image repository for Mailcatcher  | ghcr.io/nrcdt/mailcatcher  |
| image.tag                 | Tag of the Mailcatcher Docker image      | latest                     |
| image.pullPolicy          | Image pull policy                        | IfNotPresent               |
| smtp_service.port         | Port for the SMTP server                 | 25                         |
| smtp_service.type         | Kubernetes service type                  | ClusterIP                  |
| http_service.port         | Port for the web interface               | 80                         |
| http_service.type         | Kubernetes service type                  | ClusterIP                  |
| resources                 | Resource requests and limits             | {}                         |
| nodeSelector              | Node selector for scheduling             | {}                         |
| tolerations               | Tolerations for scheduling               | []                         |
| affinity                  | Pod affinity rules                       | {}                         |

#### Ingress (nginx)

| Parameter                 | Description                              | Default                    |
|---------------------------|------------------------------------------|----------------------------|
| ingress.enabled           | Enable Ingress resource                  | false                      |
| ingress.className         | Ingress class name                       | nginx                      |
| ingress.hosts             | List of ingress hosts                    | []                         |
| ingress.tls               | TLS configuration for Ingress            | []                         |
| ingress.htpasswd.enabled  | Enable htpasswd basic auth               | false                      |
| ingress.htpasswd.user     | Username for web interface               |                            |
| ingress.htpasswd.password | Password for web interface               |                            |

#### Gateway API (Envoy Gateway)

| Parameter                                    | Description                                                    | Default                  |
|----------------------------------------------|----------------------------------------------------------------|--------------------------|
| gateway.enabled                              | Enable Gateway API HTTPRoute                                   | false                    |
| gateway.annotations                          | Annotations for the HTTPRoute                                  | {}                       |
| gateway.parentRefs                           | Parent Gateway references (name, namespace, sectionName, port) | [{name: eg, namespace: envoy-gateway-system}] |
| gateway.hostnames                            | Hostnames to match on                                          | [mailcatcher.example.com] |
| gateway.oidc.enabled                         | Enable OIDC redirect flow via SecurityPolicy                   | false                    |
| gateway.oidc.provider.issuer                 | OIDC provider issuer URL                                       |                          |
| gateway.oidc.provider.authorizationEndpoint  | OIDC authorization endpoint                                    |                          |
| gateway.oidc.provider.tokenEndpoint          | OIDC token endpoint                                            |                          |
| gateway.oidc.clientID                        | OIDC client ID                                                 |                          |
| gateway.oidc.clientSecret                    | OIDC client secret (inline, chart creates Secret)              |                          |
| gateway.oidc.existingSecret                  | Name of existing Secret with key `client-secret` (takes precedence) |                     |
| gateway.oidc.redirectURL                     | OIDC redirect URL after login                                  |                          |
| gateway.oidc.scopes                          | OIDC scopes to request                                         | [openid, email, profile] |
| gateway.jwt.enabled                          | Enable JWT/JWKS validation via SecurityPolicy                  | false                    |
| gateway.jwt.providers                        | List of JWT providers (name, issuer, audiences, remoteJWKS.uri)|                          |

### Networking Modes

The chart supports two networking modes for exposing the web interface. Both can be enabled at the same time if needed.

#### 1. nginx Ingress (default)
The traditional approach using a Kubernetes `Ingress` resource with the nginx ingress controller. Supports optional htpasswd basic authentication.

```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: mailcatcher.example.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: mailcatcher-tls
      hosts:
        - mailcatcher.example.com
  htpasswd:
    enabled: true
    user: admin
    password: changeme
```

#### 2. Gateway API with Envoy Gateway
Uses a Kubernetes Gateway API `HTTPRoute` to route traffic through an existing Envoy Gateway `Gateway`. Supports OIDC browser login and/or JWT token validation via Envoy Gateway `SecurityPolicy`.

**Prerequisites:**
- [Envoy Gateway](https://gateway.envoyproxy.io/) installed in the cluster
- A `Gateway` resource already provisioned (the chart creates only the `HTTPRoute`, not the `Gateway`)
- For OIDC: a client registered with your identity provider (e.g., Keycloak, Google, Azure AD)

**Gateway API only (no auth):**
```yaml
gateway:
  enabled: true
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - mailcatcher.example.com
```

**Gateway API with OIDC login (browser-based):**
```yaml
gateway:
  enabled: true
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - mailcatcher.example.com
  oidc:
    enabled: true
    provider:
      issuer: https://keycloak.example.com/realms/myrealm
      authorizationEndpoint: https://keycloak.example.com/realms/myrealm/protocol/openid-connect/auth
      tokenEndpoint: https://keycloak.example.com/realms/myrealm/protocol/openid-connect/token
    clientID: mailcatcher
    clientSecret: my-client-secret
    redirectURL: https://mailcatcher.example.com/oauth2/callback
    scopes:
      - openid
      - email
      - profile
```

**Gateway API with JWT/JWKS validation (API access):**
```yaml
gateway:
  enabled: true
  parentRefs:
    - name: eg
      namespace: envoy-gateway-system
  hostnames:
    - mailcatcher.example.com
  jwt:
    enabled: true
    providers:
      - name: keycloak
        issuer: https://keycloak.example.com/realms/myrealm
        audiences:
          - mailcatcher
        remoteJWKS:
          uri: https://keycloak.example.com/realms/myrealm/protocol/openid-connect/certs
```

**Both OIDC and JWT can be enabled simultaneously** -- OIDC handles browser-based login while JWT validates tokens for programmatic API access.

### Accessing the Service
#### SMTP Server
Send emails to the SMTP server at service-name.namespace.svc.cluster.local:smtp-port.

#### Web Interface
Access the web interface to view captured emails:

If using a ClusterIP service:

Use kubectl port-forward to access the service locally:
```
kubectl port-forward service/mailcatcher 1080:1080 -n <your-namespace>
```
Open your browser and navigate to http://localhost:1080.
If using an Ingress resource:

Navigate to the specified hostname (e.g., http://mailcatcher.example.com).

If using Gateway API:

Navigate to the hostname configured in `gateway.hostnames`. If OIDC is enabled, you will be redirected to your identity provider for login.

### Uninstallation
To uninstall the Mailcatcher release:
```
helm uninstall mailcatcher --namespace <your-namespace>
```

### Troubleshooting
No emails are captured: Ensure your application is configured to send emails to the SMTP server at the correct address and port.

Cannot access the web interface: Verify the service type (ClusterIP, NodePort, or Ingress) and any network policies or firewalls in place.

OIDC redirect fails: Verify that the `redirectURL` matches the callback URL registered with your identity provider, and that the `clientID` and `clientSecret` are correct.

JWT validation fails: Ensure the `remoteJWKS.uri` is reachable from the Envoy Gateway pods and that the `issuer` and `audiences` match the tokens being presented.
