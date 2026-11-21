# Hands-on

The course hands-on exercises were applied **directly to the group project**, instead of in a separate
study repository: each one became a part of the platform. The table shows where each hands-on lives and who
did that part.

| Hands-on | Topic | Where it is in the project | Who |
|---|---|---|---|
| 0 | Environment, root repository and MkDocs | [`pna26/platform`](https://github.com/pna26/platform){target="_blank"} with the services as submodules in `api/` and the [documentation](https://pna26.github.io/platform/){target="_blank"} | **me** |
| 1 | Account (controller, repository, container) | [`account`](https://github.com/pna26/account){target="_blank"} + [`account-service`](https://github.com/pna26/account-service){target="_blank"} | Nicolas |
| 2 | Gateway | [`gateway-service`](https://github.com/pna26/gateway-service){target="_blank"}: routes and `GET /health-check` ([Project](projeto.md#gateway)) | **me** |
| 3 | Security (JWT, open routes, authorization) | gateway authorization filter ([Project](projeto.md#gateway)); JWT issuance in `auth-service` | **me** (gateway) and Nicolas (Auth) |
| 4 | CI/CD (Jenkins) | done with **GitHub Actions**, which the course accepts in place of Jenkins: reusable pipeline ([Project](projeto.md#cicd)) | **me** |
| 5 | Kubernetes | gateway and `platform` manifests (`lb`, Redis); EKS cluster | **me** (manifests) and Nicolas (EKS) |
| 6 | Observability (Prometheus + Grafana) | [Observability](https://pna26.github.io/platform/bottlenecks/observability/){target="_blank"} | Gabriel |
| 7 | Load balancing (Nginx) | [Load balancing](bottlenecks.md#balanceamento-de-carga) | **me** |
| 8 | Distributed cache (Redis) | [Token cache](bottlenecks.md#cache-do-token-redis) | **me** |
| 9 | Database replication and failover | not applied in the project (on EKS the database is RDS, managed by AWS) | — |
| 10 | Messaging (Kafka) | [Messaging](https://pna26.github.io/platform/bottlenecks/messaging/){target="_blank"} | Nicolas |
| 11 | Distributed tracing (traces) | not applied in the project | — |
| 12 | AI in CI/CD | not applied in the project (AI was used during development, see [Use of AI](uso-de-ia.md)) | — |

Beyond the hands-on exercises, the [vulnerability scanning](bottlenecks.md#varredura-de-vulnerabilidades) bottleneck
came from the course's bottlenecks page (Trivy and OWASP ZAP).
