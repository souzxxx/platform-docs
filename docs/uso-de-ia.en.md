# Use of AI

The course allows using AI in the implementation as long as the use is documented and each member
can explain and defend every part. The group's full record, per member, is in
[Uso de IA](https://pna26.github.io/platform/uso-de-ia/){target="_blank"}; here is the summary of my part.

**Tool:** Claude Code (Anthropic), in the terminal, with access to the organization's repositories.

**How I used it:** starting from the project board and the course material (hands-on 2, 3, 4, 7 and 8 and the
bottlenecks page), I asked the AI to implement my tasks. The AI generated the code, ran the tests and the
platform locally, and opened the PRs. I reviewed, decided what went in and checked the results.

| Part | With AI? | How it was verified |
|---|---|---|
| Gateway | Yes, starting from hands-on 2 and 3 | 20 automated tests; end-to-end smoke test in CI |
| Token cache in Redis | Yes | tested with Redis on and off; `ab` numbers |
| Nginx (load balancing) | Yes, starting from hands-on 7 | `nginx -t`; smoke test with 3 replicas |
| CI/CD pipeline and OIDC on AWS | Yes | `actionlint`; real runs on `main` publishing to ECR |
| Vulnerability scanning | Yes | the fixes came from Trivy's own findings |
| Review of teammates' PRs | Yes, reading the diffs with the AI | the decision to merge and the comments are mine |
| Documentation (this one and my pages in the group docs) | Yes | based on the professor's template; `mkdocs build --strict` |
| Presentation video and English translation | Yes: script, page recording (Playwright), editing (ffmpeg) and translation | each scene checked with frames extracted from the video; the EKS demo hides the JWT and the password; the English version keeps the Portuguese page anchors |

**What I did without AI:**

- **Project organization:** the [project board](https://github.com/orgs/pna26/projects/1){target="_blank"}
  with the 22 task cards, the split of work among the three members, the weight of each deliverable and
  the owner of each item.
- **API contract:** Gateway, Auth, Account and Exchange routes, open routes, JWT claims and lifetime
  and cookie configuration, the basis of the
  [Arquitetura](https://pna26.github.io/platform/arquitetura/){target="_blank"} page.
- **Review of the generated code** and the direction of the work: which tasks the AI did and the approval of each
  external action (repositories created, PRs, merges).

**Where the AI got it wrong and it was fixed:**

- The first benchmark with and without cache was invalid: the `TOKEN_CACHE_ENABLED` variable did not reach the
  gateway through compose, so both rounds ran with the cache. Fixed and measured again.
- Trivy aborted when `--ignorefile` pointed to a nonexistent file: the pipeline now only
  uses `.trivyignore` when it exists.
- `WebClient.Builder` is no longer auto-configured in Spring Boot 4 without `spring-boot-starter-webclient`;
  the integration test caught that.
