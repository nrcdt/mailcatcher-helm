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

#### Autoscaling

| Parameter                              | Description                                                      | Default            |
|----------------------------------------|------------------------------------------------------------------|--------------------|
| autoscaling.enabled                    | Enable HorizontalPodAutoscaler                                  | false              |
| autoscaling.minReplicas                | Minimum number of replicas                                       | 1                  |
| autoscaling.maxReplicas                | Maximum number of replicas                                       | 100                |
| autoscaling.targetCPUUtilizationPercentage | Target CPU utilization for scaling                           | 80                 |
| autoscaling.vpa.enabled                | Enable VerticalPodAutoscaler (requires VPA controller)           | false              |
| autoscaling.vpa.mode                   | VPA update mode: Off, Initial, Recreate, InPlaceOrRecreate, Auto | InPlaceOrRecreate  |
| autoscaling.vpa.minAllowed.cpu         | Minimum CPU the VPA can set                                      | 10m                |
| autoscaling.vpa.minAllowed.memory      | Minimum memory the VPA can set                                   | 50Mi               |
| autoscaling.vpa.maxAllowed.cpu         | Maximum CPU the VPA can set (defaults to resources.limits.cpu)   |                    |
| autoscaling.vpa.maxAllowed.memory      | Maximum memory the VPA can set (defaults to resources.limits.memory) |                |

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
| gateway.healthRoute.enabled                  | Create an unauthenticated HTTPRoute for external health checks | false                    |
| gateway.healthRoute.path                     | Exact path the health checker calls                            | /healthz                 |
| gateway.healthRoute.backendPath              | Path on Mailcatcher the request is rewritten to                | /                        |
| gateway.oidc.enabled                         | Enable OIDC redirect flow via SecurityPolicy                   | false                    |
| gateway.oidc.provider.issuer                 | OIDC provider issuer URL                                       |                          |
| gateway.oidc.provider.authorizationEndpoint  | OIDC authorization endpoint (optional, discovered from issuer) |                          |
| gateway.oidc.provider.tokenEndpoint          | OIDC token endpoint (optional, discovered from issuer)         |                          |
| gateway.oidc.clientID                        | OIDC client ID                                                 |                          |
| gateway.oidc.clientSecret                    | OIDC client secret (inline, chart creates Secret)              |                          |
| gateway.oidc.existingSecret                  | Name of existing Secret with key `client-secret` (takes precedence) |                     |
| gateway.oidc.redirectURL                     | OIDC redirect URL after login                                  |                          |
| gateway.oidc.logoutPath                      | Path that clears the OIDC session (e.g. `/logout`)             |                          |
| gateway.oidc.cookieDomain                    | Domain for the session cookies                                 |                          |
| gateway.oidc.forwardAccessToken              | Forward access token as `Authorization: Bearer` to the backend | false                    |
| gateway.oidc.refreshToken                    | Automatically refresh the access token                         | false                    |
| gateway.oidc.scopes                          | OIDC scopes to request                                         | [openid, email, profile] |
| gateway.oidc.resources                       | Resource indicators (RFC 8707) sent to the IdP                 | []                       |
| gateway.oidc.extra                           | Additional fields passed verbatim to `spec.oidc`               | {}                       |
| gateway.jwt.enabled                          | Enable JWT/JWKS validation via SecurityPolicy                  | false                    |
| gateway.jwt.optional                         | Allow requests without a JWT (invalid JWTs are still rejected) | false                    |
| gateway.jwt.providers                        | List of JWT providers (see below)                              |                          |
| gateway.jwt.providers[].name                 | Provider name (referenced by authorization rules)              | default                  |
| gateway.jwt.providers[].issuer               | Expected `iss` claim                                           |                          |
| gateway.jwt.providers[].audiences            | Accepted `aud` values                                          | []                       |
| gateway.jwt.providers[].remoteJWKS.uri       | JWKS endpoint URL                                              |                          |
| gateway.jwt.providers[].localJWKS            | Inline JWKS JSON (alternative to remoteJWKS)                   |                          |
| gateway.jwt.providers[].claimToHeaders       | Copy claims into request headers (`[{header, claim}]`)         | []                       |
| gateway.jwt.providers[].extractFrom          | Token location (headers/cookies/params)                        | {}                       |
| gateway.jwt.providers[].recomputeRoute       | Recompute the route after claims were copied to headers        | false                    |
| gateway.authorization.enabled                | Enable authorization rules via SecurityPolicy                  | false                    |
| gateway.authorization.defaultAction          | Action if no rule matches (`Deny` or `Allow`)                  | Deny                     |
| gateway.authorization.rules                  | Envoy Gateway authorization rules (JWT claims/scopes, clientCIDRs) | []                   |

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

**OIDC with endpoint discovery and logout:**

`authorizationEndpoint` and `tokenEndpoint` can be omitted; Envoy Gateway then discovers them via `<issuer>/.well-known/openid-configuration`.
```yaml
gateway:
  enabled: true
  hostnames:
    - mailcatcher.example.com
  oidc:
    enabled: true
    provider:
      issuer: https://keycloak.example.com/realms/myrealm
    clientID: mailcatcher
    existingSecret: mailcatcher-oidc   # Secret with key "client-secret"
    redirectURL: https://mailcatcher.example.com/oauth2/callback
    logoutPath: /logout
    refreshToken: true
```

**JWT with claims forwarded as headers and restricted to a group:**
```yaml
gateway:
  enabled: true
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
        claimToHeaders:
          - header: x-user-email
            claim: email
  authorization:
    enabled: true
    defaultAction: Deny
    rules:
      - name: allow-mailcatcher-users
        action: Allow
        principal:
          jwt:
            provider: keycloak
            claims:
              - name: groups
                valueType: StringArray
                values: ["mailcatcher-users"]
```

**Health checks through the gateway:**

With OIDC, JWT or authorization enabled, every request through the HTTPRoute must authenticate, so an upstream load balancer or uptime monitor gets a 302/401/403 instead of 200. Kubelet probes and Envoy Gateway's own backend health checks go straight to the pod and are not affected. For external checkers, enable the health route:
```yaml
gateway:
  healthRoute:
    enabled: true
    path: /healthz     # point the checker at https://mailcatcher.example.com/healthz
```
This creates a second HTTPRoute `<fullname>-health` with an `Exact` match on `path`. The SecurityPolicy only targets the main route, so this path skips authentication. Gateway API prefers exact matches over the main route's `/` prefix. The request is rewritten to `backendPath` (default `/`), which returns Mailcatcher's static UI without any messages, so the check proves the pod is reachable without exposing mail.

Values are validated against `values.schema.json` on `helm install`, `upgrade`, `lint` and `template`: types and enums, plus the fields required once a feature is enabled (e.g. `oidc.provider.issuer`, `oidc.clientID`, `oidc.redirectURL`, a client secret, or a JWKS source for each JWT provider).

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
