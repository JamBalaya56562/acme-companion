# acme-companion — Open Issue Triage

> Management / tracking document for our team. **Not intended for an upstream PR.**
> This branch exists only to persist our issue investigation outside the sandbox.

- **Upstream repo:** [nginx-proxy/acme-companion](https://github.com/nginx-proxy/acme-companion)
- **Snapshot date:** 2026-06-17 · **Last updated:** 2026-07-18
- **Scope:** the open issues investigated and triaged at the snapshot date; each issue is removed from this report once its PR merges or it is otherwise closed.
- **Method:** Each issue's full thread was read and its claim verified against the current `main` code (`app/functions.sh`, `app/entrypoint.sh`, `app/letsencrypt_service`, `app/cert_status`, `Dockerfile`, `docs/`). Classifications are evidence-based, not taken at face value.

## Progress log

Running record of what our team has actioned (most recent first):

| Date | Issue | Action | Reference |
|------|-------|--------|-----------|
| 2026-07-18 | PR #1128 | Implemented (superseding stale POC PR): standalone certs for containers with `LETSENCRYPT_HOST` but no `VIRTUAL_HOST`, via the tmpl-level detection buchdag suggested in the #1128 thread (idea maintainer-approved 2024-06); new `ACME_STANDALONE_CONTAINERS` array + `standalone_container` test + docs → PR opened | [PR #1294](https://github.com/nginx-proxy/acme-companion/pull/1294) |
| 2026-07-18 | #1221 | Replied: real cause is a wrong env var name (`ACME_CHALLENGE_TYPE`, not `ACME_CHALLENGE`) → DNS-01 never enabled, request fell through to HTTP-01 (webroot); gave the correct `ACME_CHALLENGE` + `ACMESH_DNS_API_CONFIG` (Route 53) setup | [comment](https://github.com/nginx-proxy/acme-companion/issues/1221#issuecomment-5007924909) |
| 2026-06-20 | #1006 | Implemented: `reload_nginx`/`check_nginx_proxy_container_run`/entrypoint docker-gen check loop over all label-matched IDs (multi nginx replica) + test → PR opened | [PR #1275](https://github.com/nginx-proxy/acme-companion/pull/1275) |
| 2026-06-20 | #709 | Implemented: Docker `HEALTHCHECK` (PID-file liveness for `letsencrypt_service` + docker-gen) + test + docs → PR opened | [PR #1273](https://github.com/nginx-proxy/acme-companion/pull/1273) |
| 2026-06-17 | #964 | Replied: `LETSENCRYPT_HOST` supports comma-separated multi-domain (SAN) | [comment](https://github.com/nginx-proxy/acme-companion/issues/964#issuecomment-4726839468) |

> Issues whose PR has merged or that were otherwise closed are removed from this report (per the scope above); their delivered work lives in the linked PRs.

## Classification legend

| Tag | Meaning |
|-----|---------|
| `implement` | Valid bug or feature that needs a code change |
| `docs` | Only needs a documentation update |
| `invalid` | Claim is wrong / user error / misconfiguration |
| `outdated` | Was valid once but already resolved/obsolete in current code |
| `needs-info` | Cannot be verified without more info from reporter |
| `question` | Usage/support question, no code change warranted |
| `duplicate` | Duplicate of another issue/PR |
| `out-of-scope` | Legitimate but outside the project's scope (Swarm, host networking, host OS, upstream nginx-proxy, …) |

## Summary counts

| Classification | Count | Issues |
|---|---|---|
| `implement` | 4 | #1062, #1006, #997, #709 |
| `out-of-scope` | 10 | #1197, #998, #968, #894, #704, #699, #600, #453, #258, #187 |
| `invalid` | 9 | #1200, #1098, #962, #959, #949, #933, #653, #625, #607 |
| `outdated` | 4 | #1090, #1051, #853, #242 |
| `needs-info` | 3 | #1221, #1196, #1042 |
| `question` | 2 | #964, #764 |
| `duplicate` | 1 | #367 |

---

## 🎯 Actionable work queue (`implement` + `docs`)

Ordered roughly by value/effort. `Status` is ours to maintain (TODO / WIP / PR #… / Done).

| Pri | Issue | Title | Effort | In scope? | Status |
|-----|-------|-------|--------|-----------|--------|
| 🔄 | [#709](https://github.com/nginx-proxy/acme-companion/issues/709) | Add Docker `HEALTHCHECK` | medium | ✅ (maintainer-endorsed) | **PR #1273 (open, awaiting review)** |
| 🔄 | [#1006](https://github.com/nginx-proxy/acme-companion/issues/1006) | Multi-replica nginx: loop reload over all matching container IDs | medium | ✅ (single-node) | **PR #1275 (open, awaiting review)** |
| 9 | [#997](https://github.com/nginx-proxy/acme-companion/issues/997) | Honor nginx-proxy `CERT_NAME` for generated cert filenames | medium | ✅ | TODO |
| — | [#1062](https://github.com/nginx-proxy/acme-companion/issues/1062) | LE-signed default certificate | medium | ⚠️ mostly nginx-proxy side (`DEFAULT_CERT_NAME`) | TODO |

**Suggested starters:** **#709** (PR #1273) + **#1006** (PR #1275) are in review; the next clean in-scope pick is **#997** (honor `CERT_NAME`).

---

## Detailed findings

> Grouped by classification. Each entry records the verified verdict and the recommended action.

### `implement` — needs a code change

#### #709 — Support Docker health checks
- **Created:** 2020-11 (~5.6y) · **Effort:** medium
- **Claim:** Provide a Docker `HEALTHCHECK` so `compose up --wait` can gate on readiness.
- **Verification:** **true.** No `HEALTHCHECK` in `Dockerfile`, no health script anywhere. `app/start.sh` backgrounds `letsencrypt_service` + `docker-gen` but its wait loop only monitors the docker-gen PID — if `letsencrypt_service` dies the container stays "up" while renewal silently stops.
- **Action:** **PR [#1273](https://github.com/nginx-proxy/acme-companion/pull/1273) opened (awaiting review).** `start.sh` records both PIDs under `/var/run`; new `app/healthcheck.sh` checks both with `kill -0`; `Dockerfile` adds `HEALTHCHECK` (interval 30s/timeout 5s/start-period 30s/retries 3). PID files (not `pgrep`) avoid matching docker-gen, whose args contain `letsencrypt_service_data`. Adds a `container_health` integration test + `docs/Docker-Compose.md` `--wait`/`service_healthy` note. Verified locally: status flips healthy→unhealthy on `letsencrypt_service` death.

#### #1006 — 3-container setup + nginx replicas
- **Created:** 2023-03 (~3.3y) · **Effort:** medium
- **Claim:** With ≥2 nginx replicas sharing one label, `get_nginx_proxy_container` returns concatenated IDs; reload/health-check don't handle multiple IDs and fail.
- **Verification:** **true.** `labeled_cid` (`functions.sh:271-273`) emits one ID per match (newline-separated); `check_nginx_proxy_container_run` (34-47) and `reload_nginx` (329-352) each assumed a single ID and embedded it directly in the Docker API URL → broke with concatenated IDs.
- **Action:** **PR [#1275](https://github.com/nginx-proxy/acme-companion/pull/1275) opened (awaiting review).** Both functions now `mapfile` the IDs and loop: health-check returns OK if any replica is running; reload sends SIGHUP / `docker exec` to each docker-gen and nginx replica. **Also fixes the `entrypoint.sh` startup check (review catch):** new `nginx_proxy_is_docker_gen` helper iterates the IDs so several all-in-one nginx-proxy replicas no longer fail with `can't get docker-gen container id`. Single-container behavior unchanged (1-element loop), so existing `docker_api` tests still pass. New deterministic `nginx_replicas` test covers both the separate-docker-gen (SIGHUP) and all-in-one startup paths. Only `docker_restart` (proxied app containers) remains out of scope. Locally verified: 2 replicas → both get SIGHUP, check returns 0, and the companion starts with 2 all-in-one replicas.

#### #997 — Honor `CERT_NAME` environment variable
- **Created:** 2023-02 (~3.3y) · **Effort:** medium
- **Claim:** nginx-proxy looks for `/etc/nginx/certs/$CERT_NAME.crt`; companion should name generated files after `CERT_NAME`.
- **Verification:** **true.** No `CERT_NAME` anywhere in `app/`/`docs/`. Filenames derive solely from the domain (`letsencrypt_service:143`, symlinks 50-52); the data template only reads `LETSENCRYPT_HOST`.
- **Action:** Plumb `CERT_NAME` through the data template; create `$CERT_NAME.crt/.key` symlinks when set. Needs a design decision vs `LETSENCRYPT_SINGLE_DOMAIN_CERTS`.

#### #1062 — LE-signed default certificate
- **Created:** 2023-10 (~2.6y) · **Effort:** medium (mostly nginx-proxy)
- **Claim:** The self-signed `default.crt` (`CN=acme-companion`) causes warnings when a backend is down; allow a real LE cert as the default.
- **Verification:** **true.** `entrypoint.sh:145-181` still generates a self-signed default (now gated behind `CREATE_DEFAULT_CERTIFICATE`, default false, with a deprecation warning). Symlink creation skips `default` (`letsencrypt_service:78`). No way to point default at a real cert.
- **Action:** Maintainers lean toward solving on **nginx-proxy** via a `DEFAULT_CERT_NAME` template var (designate an existing managed cert as default); companion side needs little/no change. Track upstream.

### `outdated` — already resolved / obsolete

#### #1090 — "no ACME account was found or registered"
- **2024-01 · label `status/needs-more-info`.** Caused by reusing an acme.sh **v2**-era `/etc/acme.sh` volume with v3 code (path layout changed `…/directory/account.json`). Current image ships acme.sh 3.x (`Dockerfile:7`); fresh/consistent-v3 volumes work. Path logic at `letsencrypt_service:301-333` is correct for v3. → Close as outdated; suggest regenerating the stale account folder.

#### #1051 — CURL error 28 on cert creation/renewal
- **2023-08.** Real symptom but **upstream**: known curl bug shipped in Alpine 3.19 → endless loop/timeout on EOL 3.x kernels (old Synology, etc.). Canonical tracking issue #1084. Repo now on `alpine:3.24.1`. → Close as duplicate of #1084; affected users must upgrade host kernel. (#1042 is likely the same.)

#### #853 — Certs not renewed with custom ACME endpoint
- **2021-08.** Was valid (no way to override acme.sh `--days`); **now fixed** via `ACME_RENEW_AFTER` (+ per-container `ACME_<cid>_RENEW_AFTER`), `letsencrypt_service:11,481-485`, documented. → Close; advise setting `ACME_RENEW_AFTER` below the CA's cert lifetime.

#### #242 — Support for multiple nginx proxies
- **2017-07.** Companion-side ask is **resolved**: pin each companion to its proxy via `NGINX_PROXY_CONTAINER` or labels (`functions.sh:295,305-306`, `docs/Getting-containers-IDs.md`). Multi-host-IP `VIRTUAL_HOST` routing belongs to nginx-proxy. → Close as resolved.

### `needs-info` — unverifiable without reporter data

#### #1221 — Certificates are not generated
- **2025-11.** Reporter set `ACME_CHALLENGE_TYPE=DNS-01`, but the variable acme-companion reads is `ACME_CHALLENGE` (default `HTTP-01`, `letsencrypt_service.sh:210`), so DNS-01 was never enabled — the log's `Getting webroot …` (HTTP-01 path, no `Adding txt value …`) confirms it. The `retryafter=86400 … will not retry` line is downstream: the HTTP-01 order stays Pending and ZeroSSL hands back a 24h Retry-After. DNS settings also belong in a single `ACMESH_DNS_API_CONFIG` (JSON/YAML), not separate `DNS_API`/`AWS_*` vars. → **Replied 2026-07-18** with the corrected setup. Awaiting reporter confirmation; likely closes as `invalid` (misconfiguration).

#### #1196 — Unneeded renewal makes the service go down
- **2025-05.** Premature renewal not reproduced (acme.sh decides via `--days`; restart doesn't force renew). The ~5-min outage is likely ZeroSSL issuance latency / nginx-proxy reload, not deliberate cert removal. → Ask for `DEBUG=1` logs of the triggering restart + cert timestamps.

#### #1042 — crt/key not created in certs folder
- **2023-07.** No companion logs provided; symptoms match the curl-28/#1084 hang. → Request `DEBUG=1` logs; if `error code: 28`/"Can not init api", close as duplicate of #1084.

### `question` — usage/support, no code change

#### #964 — Multiple Domain Structure Suggestion
- **2022-08.** Premise wrong: `LETSENCRYPT_HOST` **does** support comma-separated multi-domain SAN (`letsencrypt_service_data.tmpl:19`, `docs/Basic-usage.md:51`). → Close with docs pointer.

#### #764 — Cannot access the assigned container port
- **2021-03 · label `kind/question`.** By design: nginx-proxy routes by `VIRTUAL_HOST`, not per-container published ports; HTTP-01 needs 80/443. Path-based routing is an nginx-proxy concern. → Close as answered.

### `invalid` — not a defect (user/config/upstream)

- **#1200** (2025-06) — Missing `acme-challenge` location is nginx-proxy's *fallback* template (no cert yet) + a prior LE rate-limit; reporter self-resolved via v2.4.0 guidance. Cert/symlink writing is all the companion does.
- **#1098** (2024-02) — IPv6-only host can't egress because the Docker network has `EnableIPv6:false`; enable Docker IPv6. No companion IPv6 code exists. (Reporter-confirmed workaround in thread.)
- **#962** (2022-07) — WordPress redirect loops come from WP `siteurl`/`home` + nginx-proxy headers; companion emits no redirects.
- **#959** (2022-06) — Certs are **not** re-issued on restart with persistent volumes (RENEW_SKIP=2); reporters found volume-mapping typos. "Invalid in browser" / ZeroSSL 503 are mis-mount / network issues.
- **#949** (2022-04) — "Old label" complaint is obsolete (both legacy + new labels accepted, `functions.sh:282-284`); the `nginx -s reload` failure is a network/topology misconfig (app not on proxy network).
- **#933** (2022-03) — "no live upstreams"/502 is nginx-proxy networking (app & proxy not on a shared network); companion plays no role in upstream selection.
- **#653** (2020-05) — Stopping a container removes only the **symlinks** (`cleanup_links`), not the real cert files under `/etc/nginx/certs/<domain>/`; recreated on next loop. Reporter's nginx read the symlink during a race → point it at the real files.
- **#625** (2020-03 · `status/needs-more-info`) — Apex and subdomains handled identically; reporter's own test with `httpd:alpine` worked → their image wasn't serving/reachable.
- **#607** (2019-12 · `kind/failing-authorization`) — Catch-all/`default_server` overriding the challenge, or advertised AAAA without IPv6 reachability — both documented in `docs/Invalid-authorizations.md`.

### `out-of-scope` — legitimate but outside project scope

- **#1197** (2025-05) — `ssl_stapling ignored` warning: LE dropped OCSP URLs (May 2025); the `ssl_stapling` directive comes from **nginx-proxy**, not this repo. Harmless. → Redirect to nginx-proxy.
- **#998** (2023-02) — Docker Swarm not supported (maintainer-confirmed); single-container-ID model. → Close; see `docs/Getting-containers-IDs.md`.
- **#968** (2022-10) — Two-container Swarm doesn't work; three-container + docker-gen fork on a single node does (documented). User also pointed `NGINX_DOCKER_GEN_CONTAINER` at the wrong task.
- **#894** (2021-11 · `scope/arm`) — Alpine 32-bit time64 / host `libseccomp2` issue on 32-bit Raspberry Pi OS; fix is host-side (update libseccomp2 / use 64-bit OS). Not fixable in-repo.
- **#704** (2020-10) — Proposes adopting an external Swarm fork; major architectural shift away from the docker-gen-on-single-host model. → Keep as community fork.
- **#699** (2020-09 · `kind/question`) — No built-in failure-notification; partial coverage via `ACME_POST_HOOK` + LE expiry emails. First-class monitoring is outside core scope (pairs with #709).
- **#600** (2019-12) — Cert renewal fails in nginx-proxy `host` network mode; root cause is upstream Docker (moby/moby#15086). **DNS-01** (added later, `ACME_CHALLENGE=DNS-01`) is the supported workaround → small docs note worthwhile.
- **#453** (2018-10) — AWS ECS/Beanstalk task-startup failure with obsolete images/labels; ECS orchestration debugging isn't this repo's domain.
- **#258** (2017-09) — Multi-node Swarm unsupported (same-node-only caveat documented); thread consensus: use Kubernetes + cert-manager.
- **#187** (2017-03) — Cross-container SAN merging intentionally not supported (maintainer declined; "doubt it will be feasible"). Multi-domain must live on one container's `LETSENCRYPT_HOST`.

### `duplicate`

- **#367** (2018-04 · `status/duplicate`) — Certs for local `.test` domains: LE can't validate non-public domains. Duplicate of #8 (self-signed for development). → Use `docker exec` openssl workaround or mkcert.

---

## Notes & caveats

- **#709** (PR #1273) + **#1006** (PR #1275) are in review; the next clearly actionable, in-scope, low-risk item is **#997** (honor `CERT_NAME`).
- Several `invalid`/`question`/`failing-authorization` issues all reduce to the same root causes documented in `docs/Invalid-authorizations.md` (shared volumes, IPv6/AAAA reachability, app on the proxy network).
- `out-of-scope` here means "not addressable in this repo" (Swarm, host networking, host OS, or an upstream nginx-proxy/acme.sh concern) — several have maintainer statements on record.
- This is a point-in-time snapshot (2026-06-17); verify status before acting, as upstream may have moved.
