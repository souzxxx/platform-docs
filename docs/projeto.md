# Projeto

Uma plataforma de microsserviços para uma loja: o usuário se cadastra, faz login e consulta câmbio
entre moedas. Todas as requisições entram por um único ponto (o **gateway**, atrás de um balanceador
**Nginx**), que valida o token JWT e repassa a identidade do usuário para os serviços. Roda com Docker
Compose na máquina e no **EKS** (Kubernetes da AWS), com deploy pelo **GitHub Actions**.

``` mermaid
flowchart LR
    internet([Internet]) ==> lb["lb · Nginx"]
    subgraph trusted [Camada confiável]
        lb ==> gw["gateway<br/>(réplicas)"]
        gw --> auth
        gw --> account
        gw --> exchange
        gw <-->|token → id| redis[(Redis)]
        auth --> account
        account --> db[(PostgreSQL)]
        account -.->|evento| kafka[[Kafka]] -.-> notification
    end
    exchange --> api3([API de câmbio])
```

| Integrante | Responsabilidades |
|---|---|
| **Leonardo** (eu) | Gateway, CI/CD, balanceamento de carga, cache (Redis), varredura de vulnerabilidades, documentação |
| Nicolas | Auth, Account, AWS, EKS, mensageria (Kafka) |
| Gabriel | Exchange, testes de carga, observabilidade, PaaS/custos/SLA |

Arquitetura completa: [Arquitetura](https://pna26.github.io/platform/arquitetura/){target="_blank"}.

## Gateway

Repositório: [pna26/gateway-service](https://github.com/pna26/gateway-service){target="_blank"} ·
Página do grupo: [Gateway](https://pna26.github.io/platform/servicos/gateway/){target="_blank"}

Feito com **Spring Cloud Gateway** sobre **WebFlux**, a partir dos hands-on 2 e 3. O gateway:

1. **roteia** cada caminho para o serviço certo (`/auth/**`, `/accounts/**`, `/exchange/**`);
2. **autoriza**: fora das rotas abertas (cadastro, login, logout), exige o cookie com o JWT, valida o
   token no Auth (`POST /auth/solve`) e repassa o id da conta no header `id-account`;
3. **descarta** qualquer `id-account` vindo do cliente, para ninguém se passar por outra conta;
4. guarda no **Redis** o resultado da validação do token ([cache](bottlenecks.md#cache-do-token-redis));
5. expõe `GET /health-check` com o hostname da réplica, que mostra o
   [balanceamento](bottlenecks.md#balanceamento-de-carga) funcionando.

`mvn verify` roda 20 testes, também no CI a cada push: rotas abertas, cache do token (a chave é o
SHA-256 do token e o TTL nunca passa do `exp`), filtro de autorização (sem cookie → `401`, Auth fora →
`503`, cache hit não chama o Auth) e um teste que sobe o gateway de verdade contra um Auth e um Account
falsos e confere que um `id-account` forjado é descartado.

## CI/CD

Página do grupo: [CI/CD](https://pna26.github.io/platform/devops/ci-cd/){target="_blank"}

Em vez de um pipeline por repositório, escrevi **um workflow reutilizável** no `platform`
(`.github/workflows/service.yaml`) e cada serviço tem só um `ci.yaml` pequeno que o chama. Uma
correção no pipeline vale para todos.

``` mermaid
flowchart LR
    push(["push na main"]) --> build["build e testes<br/>(Maven ou pytest)"]
    build --> trivy["Trivy<br/>código e imagem"]
    trivy --> gate{"CRITICAL corrigível<br/>ou segredo?"}
    gate -->|sim| fail([falha])
    gate -->|não| img["imagem multi-arch<br/>GHCR + ECR"]
    img --> deploy["deploy no EKS<br/>(kubectl apply + rollout)"]
```

- **Sem chave da AWS no GitHub:** o job entra na AWS por **OIDC**, assumindo uma role IAM só com o que
  precisa (ECR e `eks:DescribeCluster`). Comprovado nos runs da `main`, com a imagem publicada no ECR
  ([evidência](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"}).
- **Compatível com quem ainda não tinha OIDC:** o reutilizável herda as permissões de quem chama; sem
  `id-token: write`, o pipeline continua verde e publica só no GHCR.
- O deploy no EKS liga sozinho quando a variável `EKS_CLUSTER` existe na organização (o cluster é
  apagado fora dos testes para economizar). **Comprovado em 16/11:** com o cluster no ar, o mesmo
  pipeline publicou e fez o deploy de account, auth e notification sem nenhum comando manual
  (`kubectl apply` + `rollout status` verdes,
  [runs](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"}).

O workflow `integration` do `platform` sobe a plataforma inteira com o compose em todo push e PR,
roda o smoke test (que exige pelo menos duas réplicas do gateway respondendo) e o OWASP ZAP.

## Repositório raiz e documentação

Criei a organização do trabalho e o repositório raiz como no hands-on 0:

- [`pna26/platform`](https://github.com/pna26/platform){target="_blank"} com os serviços como
  **submódulos** em `api/`, o `compose.yaml`, o Nginx e os manifestos k8s compartilhados;
- a documentação em **MkDocs Material**, publicada no GitHub Pages pelo workflow `docs`
  (`mkdocs build --strict` em todo PR, então link quebrado não entra);
- o [quadro do projeto](https://github.com/orgs/pna26/projects/1){target="_blank"} com as tarefas,
  pesos e responsáveis, e o checklist da entrega.

## Status

| Item | Situação |
|---|---|
| Gateway, CI/CD e os 3 bottlenecks | prontos e verificados no compose e no CI |
| Manifestos k8s | rodaram num cluster kind igual ao EKS (ensaio do teste com HPA, 12/11) e no EKS (16/11) |
| Deploy pelo CI no EKS | comprovado em 16/11 com account, auth e notification; o gateway e o exchange entram na próxima subida do cluster |
| Workflow `infra` no EKS | falhou em 16/11 porque o `lb` não subia sem o gateway; corrigido (`resolve` no Nginx), falta rodar de novo no cluster |
