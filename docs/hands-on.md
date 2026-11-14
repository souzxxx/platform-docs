# Hands-on

Os hands-on da disciplina foram aplicados **direto no projeto do grupo**, em vez de num repositório de
estudos separado: cada um virou uma parte da plataforma. A tabela mostra onde cada hands-on está e quem
fez essa parte.

| Hands-on | Tema | Onde está no projeto | Quem |
|---|---|---|---|
| 0 | Ambiente, repositório raiz e MkDocs | [`pna26/platform`](https://github.com/pna26/platform){target="_blank"} com os serviços como submódulos em `api/` e a [documentação](https://pna26.github.io/platform/){target="_blank"} | **eu** |
| 1 | Account (controller, repository, container) | [`account`](https://github.com/pna26/account){target="_blank"} + [`account-service`](https://github.com/pna26/account-service){target="_blank"} | Nicolas |
| 2 | Gateway | [`gateway-service`](https://github.com/pna26/gateway-service){target="_blank"}: rotas e `GET /health-check` ([Projeto](projeto.md#gateway)) | **eu** |
| 3 | Segurança (JWT, rotas abertas, autorização) | filtro de autorização do gateway ([Projeto](projeto.md#gateway)); emissão do JWT no `auth-service` | **eu** (gateway) e Nicolas (Auth) |
| 4 | CI/CD (Jenkins) | feito com **GitHub Actions**, que o curso aceita no lugar do Jenkins: pipeline reutilizável ([Projeto](projeto.md#cicd)) | **eu** |
| 5 | Kubernetes | manifestos do gateway e do `platform` (`lb`, Redis); cluster EKS | **eu** (manifestos) e Nicolas (EKS) |
| 6 | Observabilidade (Prometheus + Grafana) | [Observabilidade](https://pna26.github.io/platform/bottlenecks/observability/){target="_blank"} | Gabriel |
| 7 | Balanceamento de carga (Nginx) | [Balanceamento](bottlenecks.md#balanceamento-de-carga) | **eu** |
| 8 | Cache distribuído (Redis) | [Cache do token](bottlenecks.md#cache-do-token-redis) | **eu** |
| 9 | Replicação e failover do banco | não aplicado no projeto (no EKS o banco é o RDS, gerenciado pela AWS) | — |
| 10 | Mensageria (Kafka) | [Mensageria](https://pna26.github.io/platform/bottlenecks/messaging/){target="_blank"} | Nicolas |
| 11 | Rastreamento distribuído (traces) | não aplicado no projeto | — |
| 12 | IA no CI/CD | não aplicado no projeto (a IA foi usada no desenvolvimento, ver [Uso de IA](uso-de-ia.md)) | — |

Além dos hands-on, o bottleneck de [varredura de vulnerabilidades](bottlenecks.md#varredura-de-vulnerabilidades)
saiu da página de bottlenecks da disciplina (Trivy e OWASP ZAP).
