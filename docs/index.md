# Leonardo · Platform 2026.2

???+ info inline end "Disciplina"

    Plataformas, Microsserviços, DevOps e APIs
    Insper · 2026.2

Documentação individual de **Leonardo de Souza Lima e Silva**
([@souzxxx](https://github.com/souzxxx){target="_blank"}) na disciplina. O projeto foi feito em
grupo (pna26: Leonardo, Nicolas e Gabriel); aqui estão a minha parte, os bottlenecks que eu
implementei e os links de tudo o que foi usado.

**Documentação completa do grupo:** [pna26.github.io/platform](https://pna26.github.io/platform/){target="_blank"}

## Vídeo de apresentação (1–2 min)

Vídeo do grupo, sem narração e com legendas: arquitetura, CI/CD, a plataforma rodando no EKS, os
bottlenecks, o teste de carga com HPA e os custos. Roteiro em
[Apresentação](https://pna26.github.io/platform/apresentacao/){target="_blank"}.

<!-- video-embed: <video controls preload="metadata" width="100%" src="assets/apresentacao-pna26.mp4"></video> -->

## O que eu fiz

| Entrega | O quê | Detalhes |
|---|---|---|
| Microsserviço **Gateway** | Ponto único de entrada: rotas, validação do token no Auth, cache no Redis | [Projeto](projeto.md#gateway) |
| **CI/CD** (GitHub Actions) | Pipeline reutilizável usado por todos os serviços: build, testes, Trivy, imagens no GHCR e no ECR, deploy no EKS | [Projeto](projeto.md#cicd) |
| Bottleneck: **balanceamento de carga** | Nginx na frente das réplicas do gateway | [Meus bottlenecks](bottlenecks.md#balanceamento-de-carga) |
| Bottleneck: **cache (Redis)** | Validação do token guardada no Redis | [Meus bottlenecks](bottlenecks.md#cache-do-token-redis) |
| Bottleneck: **varredura de vulnerabilidades** | Trivy no pipeline e OWASP ZAP na plataforma rodando | [Meus bottlenecks](bottlenecks.md#varredura-de-vulnerabilidades) |
| **Documentação** | Repositório raiz `platform`, MkDocs, submódulos e a organização do projeto | [Projeto](projeto.md#repositorio-raiz-e-documentacao) |

## Repositórios usados

| Repositório | O que é | Responsável |
|---|---|---|
| [pna26/platform](https://github.com/pna26/platform){target="_blank"} | Repositório raiz: documentação, `compose.yaml`, Nginx, manifestos k8s, pipeline reutilizável e os serviços como submódulos | Leonardo |
| [pna26/gateway-service](https://github.com/pna26/gateway-service){target="_blank"} | API gateway (Spring Cloud Gateway) | Leonardo |
| [pna26/auth](https://github.com/pna26/auth){target="_blank"} | Interface (Feign) e DTOs do Auth | Nicolas |
| [pna26/auth-service](https://github.com/pna26/auth-service){target="_blank"} | Implementação do Auth (JWT) | Nicolas |
| [pna26/account](https://github.com/pna26/account){target="_blank"} | Interface (Feign) e DTOs do Account | Nicolas |
| [pna26/account-service](https://github.com/pna26/account-service){target="_blank"} | Implementação do Account (PostgreSQL + Flyway) | Nicolas |
| [pna26/exchange-service](https://github.com/pna26/exchange-service){target="_blank"} | Câmbio (Python/FastAPI) | Gabriel |
| [pna26/notification-service](https://github.com/pna26/notification-service){target="_blank"} | Consumidor Kafka: e-mail de boas-vindas | Nicolas |
| [souzxxx/platform-docs](https://github.com/souzxxx/platform-docs){target="_blank"} | Esta documentação individual | Leonardo |

Quadro de tarefas do grupo: [Projeto Platform 2026.2](https://github.com/orgs/pna26/projects/1){target="_blank"}.
