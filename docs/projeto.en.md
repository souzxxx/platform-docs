# Project

A microservices platform for a store: the user signs up, logs in and checks exchange rates
between currencies. Every request comes in through a single entry point (the **gateway**, behind an
**Nginx** load balancer), which validates the JWT token and forwards the user's identity to the services. It runs with Docker
Compose locally and on **EKS** (AWS Kubernetes), with deployment through **GitHub Actions**.

``` mermaid
flowchart LR
    internet([Internet]) ==> lb["lb · Nginx"]
    subgraph trusted [Trusted layer]
        lb ==> gw["gateway<br/>(replicas)"]
        gw --> auth
        gw --> account
        gw --> exchange
        gw <-->|token → id| redis[(Redis)]
        auth --> account
        account --> db[(PostgreSQL)]
        account -.->|event| kafka[[Kafka]] -.-> notification
    end
    exchange --> api3([Exchange rate API])
```

| Member | Responsibilities |
|---|---|
| **Leonardo** (me) | Gateway, CI/CD, load balancing, cache (Redis), vulnerability scanning, documentation |
| Nicolas | Auth, Account, AWS, EKS, messaging (Kafka) |
| Gabriel | Exchange, load testing, observability, PaaS/costs/SLA |

Full architecture: [Arquitetura](https://pna26.github.io/platform/arquitetura/){target="_blank"}.

## Gateway { #gateway }

Repository: [pna26/gateway-service](https://github.com/pna26/gateway-service){target="_blank"} ·
Group page: [Gateway](https://pna26.github.io/platform/servicos/gateway/){target="_blank"}

Built with **Spring Cloud Gateway** on **WebFlux**, starting from hands-on 2 and 3. The gateway:

1. **routes** each path to the right service (`/auth/**`, `/accounts/**`, `/exchange/**`);
2. **authorizes**: outside the open routes (sign-up, login, logout), it requires the cookie with the JWT, validates the
   token against Auth (`POST /auth/solve`) and forwards the account id in the `id-account` header;
3. **discards** any `id-account` coming from the client, so no one can impersonate another account;
4. stores the token validation result in **Redis** ([cache](bottlenecks.md#cache-do-token-redis));
5. exposes `GET /health-check` with the replica's hostname, which shows
   [load balancing](bottlenecks.md#balanceamento-de-carga) working.

`mvn verify` runs 20 tests, also in CI on every push: open routes, token cache (the key is the
SHA-256 of the token and the TTL never exceeds `exp`), authorization filter (no cookie → `401`, Auth down →
`503`, a cache hit does not call Auth) and a test that starts the real gateway against a fake Auth and a fake
Account and checks that a forged `id-account` is discarded.

## CI/CD { #cicd }

Group page: [CI/CD](https://pna26.github.io/platform/devops/ci-cd/){target="_blank"}

Instead of one pipeline per repository, I wrote **one reusable workflow** in `platform`
(`.github/workflows/service.yaml`) and each service has only a small `ci.yaml` that calls it. A
fix in the pipeline applies to all of them.

``` mermaid
flowchart LR
    push(["push to main"]) --> build["build and tests<br/>(Maven or pytest)"]
    build --> trivy["Trivy<br/>code and image"]
    trivy --> gate{"fixable CRITICAL<br/>or secret?"}
    gate -->|yes| fail([fail])
    gate -->|no| img["multi-arch image<br/>GHCR + ECR"]
    img --> deploy["deploy to EKS<br/>(kubectl apply + rollout)"]
```

- **No AWS key on GitHub:** the job signs in to AWS through **OIDC**, assuming an IAM role with only what it
  needs (ECR and `eks:DescribeCluster`). Proven in the `main` runs, with the image published to ECR
  ([evidence](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"}).
- **Compatible with repositories that did not have OIDC yet:** the reusable workflow inherits the caller's permissions; without
  `id-token: write`, the pipeline stays green and publishes to GHCR only.
- Deployment to EKS turns on by itself when the `EKS_CLUSTER` variable exists in the organization (the cluster is
  deleted outside of testing to save money). **Proven on 16/11:** with the cluster up, the same
  pipeline published and deployed account, auth and notification without any manual command
  (`kubectl apply` + `rollout status` green,
  [runs](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"}).

The `integration` workflow in `platform` brings up the whole platform with compose on every push and PR,
runs the smoke test (which requires at least two gateway replicas responding) and OWASP ZAP.

## Root repository and documentation { #repositorio-raiz-e-documentacao }

I set up the work organization and the root repository as in hands-on 0:

- [`pna26/platform`](https://github.com/pna26/platform){target="_blank"} with the services as
  **submodules** in `api/`, the `compose.yaml`, Nginx and the shared k8s manifests;
- the documentation in **MkDocs Material**, published to GitHub Pages by the `docs` workflow
  (`mkdocs build --strict` on every PR, so broken links don't get in);
- the [project board](https://github.com/orgs/pna26/projects/1){target="_blank"} with the tasks,
  weights and owners, and the delivery checklist.

## Status { #status }

| Item | Status |
|---|---|
| Gateway, CI/CD and the 3 bottlenecks | done and verified in compose and in CI |
| k8s manifests | ran on a kind cluster matching EKS (dry run of the HPA test, 12/11) and on EKS (16/11) |
| Deployment to EKS through CI | proven on 16/11 with account, auth and notification; on 20/11 the cluster came up again with the CI deployment and the platform answered through the ELB (gateway with 2 replicas, sign-up, login, 401 and RBAC in the video demo) |
| `infra` workflow on EKS | failed on 16/11 because the `lb` would not start without the gateway; fixed with `resolve` in Nginx: on 20/11 the `lb` answered through the ELB in front of the gateway |
| Load test with HPA on EKS | 19/11 (Gabriel): gateway from 2 → 6 replicas, 0 % failed HTTP requests, p95 254 ms ([result](https://pna26.github.io/platform/devops/load-testing/#resultado-no-eks){target="_blank"}); improvement for my gateway: graceful shutdown when replicas are removed |
