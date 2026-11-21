# Leonardo · Platform 2026.2

???+ info inline end "Course"

    Platforms, Microservices, DevOps and APIs
    Insper · 2026.2

Individual documentation of **Leonardo de Souza Lima e Silva**
([@souzxxx](https://github.com/souzxxx){target="_blank"}) for the course. The project was done as a
group (pna26: Leonardo, Nicolas and Gabriel); here you will find my part, the bottlenecks I
implemented and links to everything that was used.

**Full group documentation:** [pna26.github.io/platform](https://pna26.github.io/platform/){target="_blank"}

## Presentation video (1–2 min) { #video-de-apresentacao-12-min }

The group's video, with no narration and with captions: architecture, CI/CD, the platform running on
EKS, the bottlenecks, the load test with HPA and the costs. Script in
[Apresentação](https://pna26.github.io/platform/apresentacao/){target="_blank"}.

<video controls preload="metadata" width="100%" src="../assets/apresentacao-pna26.mp4"></video>

## What I did { #o-que-eu-fiz }

| Deliverable | What | Details |
|---|---|---|
| **Gateway** microservice | Single entry point: routes, token validation against Auth, caching in Redis | [Project](projeto.md#gateway) |
| **CI/CD** (GitHub Actions) | Reusable pipeline used by every service: build, tests, Trivy, images on GHCR and ECR, deploy to EKS | [Project](projeto.md#cicd) |
| Bottleneck: **load balancing** | Nginx in front of the gateway replicas | [My bottlenecks](bottlenecks.md#balanceamento-de-carga) |
| Bottleneck: **cache (Redis)** | Token validation stored in Redis | [My bottlenecks](bottlenecks.md#cache-do-token-redis) |
| Bottleneck: **vulnerability scanning** | Trivy in the pipeline and OWASP ZAP against the running platform | [My bottlenecks](bottlenecks.md#varredura-de-vulnerabilidades) |
| **Documentation** | Root `platform` repository, MkDocs, submodules and the project organization | [Project](projeto.md#repositorio-raiz-e-documentacao) |

## Repositories used { #repositorios-usados }

| Repository | What it is | Owner |
|---|---|---|
| [pna26/platform](https://github.com/pna26/platform){target="_blank"} | Root repository: documentation, `compose.yaml`, Nginx, k8s manifests, reusable pipeline and the services as submodules | Leonardo |
| [pna26/gateway-service](https://github.com/pna26/gateway-service){target="_blank"} | API gateway (Spring Cloud Gateway) | Leonardo |
| [pna26/auth](https://github.com/pna26/auth){target="_blank"} | Auth interface (Feign) and DTOs | Nicolas |
| [pna26/auth-service](https://github.com/pna26/auth-service){target="_blank"} | Auth implementation (JWT) | Nicolas |
| [pna26/account](https://github.com/pna26/account){target="_blank"} | Account interface (Feign) and DTOs | Nicolas |
| [pna26/account-service](https://github.com/pna26/account-service){target="_blank"} | Account implementation (PostgreSQL + Flyway) | Nicolas |
| [pna26/exchange-service](https://github.com/pna26/exchange-service){target="_blank"} | Currency exchange (Python/FastAPI, Central Bank of Brazil PTAX) | Gabriel |
| [pna26/notification-service](https://github.com/pna26/notification-service){target="_blank"} | Kafka consumer: welcome e-mail | Nicolas |
| [souzxxx/platform-docs](https://github.com/souzxxx/platform-docs){target="_blank"} | This individual documentation | Leonardo |

Group task board: [Projeto Platform 2026.2](https://github.com/orgs/pna26/projects/1){target="_blank"}.
