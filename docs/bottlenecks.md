# My bottlenecks

I implemented three of the project's bottlenecks. Each one has a full page (problem, solution,
code, evidence) in the group documentation; here is the summary with the number that shows the effect.

| Bottleneck | Number that shows the effect | Full page |
|---|---|---|
| Load balancing | 15 requests split 5/5/5 across 3 replicas, with the 3rd joining without restarting Nginx | [Load balancing](https://pna26.github.io/platform/bottlenecks/load-balancing/){target="_blank"} |
| Token cache (Redis) | 3,001 authenticated requests → **1** call to Auth | [Cache](https://pna26.github.io/platform/bottlenecks/cache/){target="_blank"} |
| Vulnerability scanning | final image with **0** HIGH/CRITICAL; the pipeline fails on a fixable CRITICAL CVE | [Scanning](https://pna26.github.io/platform/bottlenecks/vulnerability-scanning/){target="_blank"} |

## Load balancing { #balanceamento-de-carga }

**Problem:** with a single gateway, the whole platform depends on one process: if it goes down,
nothing responds; if it saturates, there is no way to scale.

**Solution:** an **Nginx** (`lb`) is the only exposed service and distributes requests across several
gateway replicas (round robin). Beyond hands-on 7:

- `server gateway:8080 resolve` with Docker's DNS re-resolved every 10 s: `docker compose up
  --scale gateway=N` adds new replicas to the load balancer **without** `nginx -s reload`;
- on Kubernetes, `keepalive_time 10s`: the Service balances **connections**, not requests, and without
  this limit the replicas created by the HPA took ≈ 1.5 min to receive load (≈ 35–40 s with it, in
  the dry run of the HPA test);
- on Kubernetes, the gateway is also resolved at runtime (kube-dns `resolver` +
  `resolve`): on EKS the `lb` starts before the gateway and, without this, Nginx crashed with *host not found*
  and blocked the `infra` workflow on 16/11. Now it becomes ready, responds `502` until the gateway exists
  and starts forwarding on its own
  ([evidence](https://pna26.github.io/platform/devops/ci-cd/#evidencia){target="_blank"});
- `/gateway/actuator/**` blocked from the outside: metrics stay on the internal network only.

**Evidence:** with 3 replicas, 15 requests through the `lb` are split evenly, and the third replica
joined while compose was already running:

``` bash
$ docker compose up -d --scale gateway=3 --no-recreate
$ for i in $(seq 1 15); do curl -s localhost:8080/health-check; echo; done | sort | uniq -c
   5 {"hostname":"112e1c20be21"}
   5 {"hostname":"5f9dceeefb1f"}
   5 {"hostname":"c98aacd33a1e"}
```

The same test runs in CI (`scripts/smoke-test.sh`) and fails if fewer than two replicas respond.

**On EKS:** on 20/11, 8 calls to `/health-check` through the ELB were answered by **2 gateway
replicas** (video demo). In the HPA test of 19/11 the gateway went from 2 to 6 replicas with no failed
HTTP request ([result](https://pna26.github.io/platform/devops/load-testing/#resultado-no-eks){target="_blank"}).

!!! warning "To improve: graceful shutdown"

    When scaling down from 6 to 2 replicas, 2 old gateway pods ended in `Error` (`0/1`): exit code 143
    (SIGTERM) from the JVM. No request was lost because the scale-down came after the load, but the
    gateway has no graceful shutdown. Next step: `server.shutdown: graceful`, a `preStop` with `sleep 5`
    so the pod leaves the Service before SIGTERM, and repeating the scale-down with k6 running.

## Token cache (Redis) { #cache-do-token-redis }

**Problem:** every authenticated request made the gateway call Auth to validate the token. With
several gateway replicas, a local cache would not be shared.

**Solution:** the gateway stores the result of `POST /auth/solve` (the account id) in **Redis**:

- the key is the **SHA-256 of the token**, never the token itself;
- the TTL never exceeds the JWT's `exp`: an expired token does not stay valid through the cache;
- if Redis goes down, the gateway keeps validating against Auth (it only logs the error), without taking anything down.

**Result** (`ab -n 3000 -c 20` on a protected route, 2 gateway replicas):

| Metric | Value |
|---|---|
| Authenticated requests | 3,001 |
| Served by Redis (`hit`) | **3,000** |
| Calls to Auth (`miss`) | **1** |

Auth was taken out of the path of 99.97 % of the requests. With Redis stopped, the gateway kept
responding `200` with a valid token and `401` without a token.

## Vulnerability scanning { #varredura-de-vulnerabilidades }

**Problem:** dependencies, base images and manifests with known flaws reach production without
anyone noticing.

**Solution:** three automatic layers, without relying on someone remembering to run them:

| Layer | Tool | Where it runs |
|---|---|---|
| Code (dependencies, secrets, Dockerfile, k8s) | Trivy | every service's pipeline |
| Final image | Trivy | every service's pipeline, before the push |
| Running application | OWASP ZAP baseline | `integration` workflow, through Nginx, like a user |

The pipeline **fails** on a CRITICAL vulnerability that has a fix or on a secret in the code; HIGH and
MEDIUM go to GitHub's *Security* tab without blocking.

**What the scan found and was fixed:**

| Finding | Fix |
|---|---|
| Image with the full JDK | `eclipse-temurin:25-jre`: fewer packages, **0 HIGH/CRITICAL** |
| Container running as root | `USER 1001` in the Dockerfile |
| Writable file system | `readOnlyRootFilesystem: true` + `emptyDir` on `/tmp` |
| Default `securityContext`, no seccomp | `runAsNonRoot`, `seccompProfile: RuntimeDefault`, `capabilities.drop: [ALL]` |

After the fixes: 0 findings in the code and 0 HIGH/CRITICAL in the image.
