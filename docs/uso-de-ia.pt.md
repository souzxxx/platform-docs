# Uso de IA

A disciplina permite usar IA na implementação desde que o uso seja documentado e que cada integrante
saiba explicar e defender todas as partes. O registro completo do grupo, por integrante, está em
[Uso de IA](https://pna26.github.io/platform/uso-de-ia/){target="_blank"}; aqui está o resumo da minha parte.

**Ferramenta:** Claude Code (Anthropic), no terminal, com acesso aos repositórios da organização.

**Como usei:** a partir do quadro do projeto e do material do curso (hands-on 2, 3, 4, 7 e 8 e a página
de bottlenecks), pedi para a IA implementar as minhas tarefas. A IA gerou o código, rodou os testes e a
plataforma localmente, e abriu os PRs. Eu revisei, decidi o que entrava e conferi os resultados.

| Parte | Com IA? | Como foi verificado |
|---|---|---|
| Gateway | Sim, a partir do hands-on 2 e 3 | 20 testes automatizados; smoke test de ponta a ponta no CI |
| Cache do token no Redis | Sim | testado com o Redis ligado e desligado; números do `ab` |
| Nginx (balanceamento) | Sim, a partir do hands-on 7 | `nginx -t`; smoke test com 3 réplicas |
| Pipeline de CI/CD e OIDC na AWS | Sim | `actionlint`; runs reais na `main` publicando no ECR |
| Varredura de vulnerabilidades | Sim | as correções vieram dos próprios achados do Trivy |
| Revisão dos PRs dos colegas | Sim, leitura dos diffs com a IA | a decisão de mergear e os comentários são meus |
| Documentação (esta e as minhas páginas do grupo) | Sim | baseada no template do professor; `mkdocs build --strict` |
| Vídeo de apresentação e tradução para o inglês | Sim: roteiro, gravação das páginas (Playwright), montagem (ffmpeg) e tradução | cada cena conferida por quadros extraídos do vídeo; a demonstração no EKS esconde o JWT e a senha; a versão em inglês mantém as âncoras das páginas em português |

**O que eu fiz sem IA:**

- **Organização do projeto:** o [quadro do projeto](https://github.com/orgs/pna26/projects/1){target="_blank"}
  com os 22 cards de tarefa, a divisão do trabalho entre os três integrantes, o peso de cada entrega e
  o responsável por cada item.
- **Contrato da API:** rotas do Gateway, Auth, Account e Exchange, rotas abertas, claims e duração do
  JWT e configuração do cookie, base da página de
  [Arquitetura](https://pna26.github.io/platform/arquitetura/){target="_blank"}.
- **Revisão do código gerado** e a direção do trabalho: quais tarefas a IA fazia e a aprovação de cada
  ação externa (repositórios criados, PRs, merges).

**Onde a IA errou e foi corrigido:**

- O primeiro benchmark com e sem cache saiu inválido: a variável `TOKEN_CACHE_ENABLED` não chegava ao
  gateway pelo compose, então as duas rodadas foram com cache. Corrigido e medido de novo.
- O Trivy abortava quando o `--ignorefile` apontava para um arquivo inexistente: o pipeline passou a só
  usar o `.trivyignore` quando ele existe.
- O `WebClient.Builder` não vem mais automático no Spring Boot 4 sem o `spring-boot-starter-webclient`;
  o teste de integração pegou isso.
