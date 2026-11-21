# Meus bottlenecks

Implementei três dos bottlenecks do projeto. Cada um tem a página completa (problema, solução,
código, evidência) na documentação do grupo; aqui está o resumo com o número que mostra o efeito.

| Bottleneck | Número que mostra o efeito | Página completa |
|---|---|---|
| Balanceamento de carga | 15 requisições divididas 5/5/5 entre 3 réplicas, com a 3ª entrando sem reiniciar o Nginx | [Balanceamento](https://pna26.github.io/platform/bottlenecks/load-balancing/){target="_blank"} |
| Cache do token (Redis) | 3.001 requisições autenticadas → **1** chamada ao Auth | [Cache](https://pna26.github.io/platform/bottlenecks/cache/){target="_blank"} |
| Varredura de vulnerabilidades | imagem final com **0** HIGH/CRITICAL; o pipeline falha com CVE CRITICAL corrigível | [Varredura](https://pna26.github.io/platform/bottlenecks/vulnerability-scanning/){target="_blank"} |

## Balanceamento de carga

**Problema:** com um único gateway, a plataforma inteira depende de um processo: se ele cai, nada
responde; se satura, não há como crescer.

**Solução:** um **Nginx** (`lb`) é o único serviço exposto e distribui as requisições entre várias
réplicas do gateway (round robin). Além do hands-on 7:

- `server gateway:8080 resolve` com o DNS do Docker re-resolvido a cada 10 s: `docker compose up
  --scale gateway=N` coloca réplicas novas no balanceamento **sem** `nginx -s reload`;
- no Kubernetes, `keepalive_time 10s`: o Service balanceia **conexões**, não requisições, e sem esse
  limite as réplicas criadas pelo HPA levavam ≈ 1,5 min para receber carga (≈ 35–40 s com ele, no
  ensaio do teste com HPA);
- no Kubernetes, o gateway também é resolvido em tempo de execução (`resolver` do kube-dns +
  `resolve`): no EKS o `lb` sobe antes do gateway e, sem isso, o Nginx caía com *host not found*
  e travou o workflow `infra` em 16/11. Agora ele fica pronto, responde `502` até o gateway existir
  e começa a encaminhar sozinho
  ([evidência](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"});
- `/gateway/actuator/**` bloqueado para fora: as métricas ficam só na rede interna.

**Evidência:** com 3 réplicas, 15 requisições pelo `lb` se dividem igualmente, e a terceira réplica
entrou com o compose já rodando:

``` bash
$ docker compose up -d --scale gateway=3 --no-recreate
$ for i in $(seq 1 15); do curl -s localhost:8080/health-check; echo; done | sort | uniq -c
   5 {"hostname":"112e1c20be21"}
   5 {"hostname":"5f9dceeefb1f"}
   5 {"hostname":"c98aacd33a1e"}
```

O mesmo teste roda no CI (`scripts/smoke-test.sh`) e falha se menos de duas réplicas responderem.

**No EKS:** em 20/11, 8 chamadas ao `/health-check` pelo ELB foram respondidas por **2 réplicas** do
gateway (demonstração do vídeo). No teste com HPA de 19/11 o gateway foi de 2 para 6 réplicas sem
nenhuma requisição HTTP com erro ([resultado](https://pna26.github.io/platform/devops/load-testing/#resultado-no-eks){target="_blank"}).

!!! warning "Ponto a melhorar: encerramento gracioso"

    Na redução de 6 para 2 réplicas, 2 pods antigos do gateway terminaram em `Error` (`0/1`): o código
    143 (SIGTERM) da JVM. Nenhuma requisição foi perdida porque a redução veio depois da carga, mas o
    gateway não tem encerramento gracioso. Próximo passo: `server.shutdown: graceful`, um `preStop` com
    `sleep 5` para o pod sair do Service antes do SIGTERM, e repetir a redução com o k6 rodando.

## Cache do token (Redis)

**Problema:** toda requisição autenticada fazia o gateway chamar o Auth para validar o token. Com
várias réplicas do gateway, um cache local não seria compartilhado.

**Solução:** o gateway guarda no **Redis** o resultado de `POST /auth/solve` (o id da conta):

- a chave é o **SHA-256 do token**, nunca o token em si;
- o TTL nunca passa do `exp` do JWT: um token vencido não continua valendo pelo cache;
- se o Redis cair, o gateway segue validando no Auth (só registra o erro), sem derrubar ninguém.

**Resultado** (`ab -n 3000 -c 20` numa rota protegida, 2 réplicas do gateway):

| Métrica | Valor |
|---|---|
| Requisições autenticadas | 3.001 |
| Resolvidas pelo Redis (`hit`) | **3.000** |
| Chamadas ao Auth (`miss`) | **1** |

O Auth saiu do caminho de 99,97 % das requisições. Com o Redis parado, o gateway continuou
respondendo `200` com token válido e `401` sem token.

## Varredura de vulnerabilidades

**Problema:** dependências, imagens base e manifestos com falhas conhecidas chegam à produção sem
ninguém ver.

**Solução:** três camadas automáticas, sem depender de alguém lembrar de rodar:

| Camada | Ferramenta | Onde roda |
|---|---|---|
| Código (dependências, segredos, Dockerfile, k8s) | Trivy | pipeline de todo serviço |
| Imagem final | Trivy | pipeline de todo serviço, antes do push |
| Aplicação rodando | OWASP ZAP baseline | workflow `integration`, pelo Nginx, como um usuário |

O pipeline **falha** com vulnerabilidade CRITICAL que tem correção ou com segredo no código; HIGH e
MEDIUM vão para a aba *Security* do GitHub sem travar.

**O que a varredura achou e foi corrigido:**

| Achado | Correção |
|---|---|
| Imagem com o JDK completo | `eclipse-temurin:25-jre`: menos pacotes, **0 HIGH/CRITICAL** |
| Container rodando como root | `USER 1001` no Dockerfile |
| Sistema de arquivos gravável | `readOnlyRootFilesystem: true` + `emptyDir` em `/tmp` |
| `securityContext` padrão, sem seccomp | `runAsNonRoot`, `seccompProfile: RuntimeDefault`, `capabilities.drop: [ALL]` |

Depois das correções: 0 achados no código e 0 HIGH/CRITICAL na imagem.
