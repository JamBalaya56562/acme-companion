# acme-companion — Open Issue Triage

> Management / tracking document for our team. **Not intended for an upstream PR.**
> This branch exists only to persist our issue investigation outside the sandbox.

- **Upstream repo:** [nginx-proxy/acme-companion](https://github.com/nginx-proxy/acme-companion)
- **Snapshot date:** 2026-06-17 · **Last updated:** 2026-06-20
- **Open issues investigated:** 41 (all open issues at snapshot; since closed: #918 via PR #1266, #710 via PR #1267, #754 via PR #1265, #1191 via PR #1268, #881 via PR #1270, #664 via PR #1272)
- **Method:** Each issue's full thread was read and its claim verified against the current `main` code (`app/functions.sh`, `app/entrypoint.sh`, `app/letsencrypt_service`, `app/cert_status`, `Dockerfile`, `docs/`). Classifications are evidence-based, not taken at face value.

## Progress log

Running record of what our team has actioned (most recent first):

| Date | Issue | Action | Reference |
|------|-------|--------|-----------|
| 2026-06-20 | #1006 | Implemented: `reload_nginx`/`check_nginx_proxy_container_run`/entrypoint docker-gen check loop over all label-matched IDs (multi nginx replica) + test → PR opened | [PR #1275](https://github.com/nginx-proxy/acme-companion/pull/1275) |
| 2026-06-20 | #664 | **PR #1272 merged → issue closed.** Compose examples rewritten (no `version:`, no `volumes_from`, `label` detection; docker-gen also mounts `certs:ro`) | [PR #1272](https://github.com/nginx-proxy/acme-companion/pull/1272) |
| 2026-06-20 | #956 | Implemented: skip destructive `cleanup_links` until docker-gen data exists (keeps valid cert symlinks on restart) + test → PR opened | [PR #1274](https://github.com/nginx-proxy/acme-companion/pull/1274) |
| 2026-06-20 | #709 | Implemented: Docker `HEALTHCHECK` (PID-file liveness for `letsencrypt_service` + docker-gen) + test + docs → PR opened | [PR #1273](https://github.com/nginx-proxy/acme-companion/pull/1273) |
| 2026-06-20 | #664 | Implemented: rewrote Compose examples (no `version:`, no `volumes_from`, `label` detection, shared `html` rw) → PR opened | [PR #1272](https://github.com/nginx-proxy/acme-companion/pull/1272) |
| 2026-06-19 | #881 | **PR #1270 merged → issue closed.** `cert_status` no longer prints `EXPIRED` for a valid leaf when only the chain fails to verify | [PR #1270](https://github.com/nginx-proxy/acme-companion/pull/1270) |
| 2026-06-19 | #1191 | **PR #1268 merged → issue closed.** Per-container `LETSENCRYPT_<cid>_RENEW_PRIVATE_KEYS` (falls back to the global) | [PR #1268](https://github.com/nginx-proxy/acme-companion/pull/1268) |
| 2026-06-18 | #1191 | Implemented: per-container `LETSENCRYPT_RENEW_PRIVATE_KEYS` → PR opened (rebased onto `main`, mergeable — awaiting review) | [PR #1268](https://github.com/nginx-proxy/acme-companion/pull/1268) |
| 2026-06-18 | #754 | **PR #1265 merged → issue closed.** `docker_api` now supports TLS-protected `tcp://` Docker hosts (`DOCKER_TLS_VERIFY`/`DOCKER_CERT_PATH`) | [PR #1265](https://github.com/nginx-proxy/acme-companion/pull/1265) |
| 2026-06-18 | #710 | **PR #1267 merged → issue closed.** Standalone challenge config now adds `listen [::]:80;` when `ENABLE_IPV6=true` | [PR #1267](https://github.com/nginx-proxy/acme-companion/pull/1267) |
| 2026-06-18 | #710 | Implemented: IPv6 standalone listener gated on `ENABLE_IPV6` → PR opened | [PR #1267](https://github.com/nginx-proxy/acme-companion/pull/1267) |
| 2026-06-18 | #918 | **PR #1266 merged → issue closed.** Routes acme.sh `--log` to `/dev/stderr` (not `/dev/null`) under `DEBUG=1` | [PR #1266](https://github.com/nginx-proxy/acme-companion/pull/1266) |
| 2026-06-17 | #918 | Implemented: gate acme.sh `--log` on `DEBUG=1` → PR opened | [PR #1266](https://github.com/nginx-proxy/acme-companion/pull/1266) |
| 2026-06-17 | #1000 | Replied: diagnosed Cloudflare/IPv6 AAAA challenge failure, recommended DNS-01 | [comment](https://github.com/nginx-proxy/acme-companion/issues/1000#issuecomment-4726889966) |
| 2026-06-17 | #964 | Replied: `LETSENCRYPT_HOST` supports comma-separated multi-domain (SAN) | [comment](https://github.com/nginx-proxy/acme-companion/issues/964#issuecomment-4726839468) |
| 2026-06-17 | #754 | Implemented: Docker API TLS client-cert support → PR opened | [PR #1265](https://github.com/nginx-proxy/acme-companion/pull/1265) |

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
| `implement` | 10 | #1191, #1062, #1006, #997, #956, #918, #881, #754, #710, #709 |
| `out-of-scope` | 10 | #1197, #998, #968, #894, #704, #699, #600, #453, #258, #187 |
| `invalid` | 9 | #1200, #1098, #962, #959, #949, #933, #653, #625, #607 |
| `outdated` | 4 | #1090, #1051, #853, #242 |
| `needs-info` | 3 | #1221, #1196, #1042 |
| `question` | 3 | #1000, #964, #764 |
| `docs` | 1 | #664 |
| `duplicate` | 1 | #367 |

---

## 🎯 Actionable work queue (`implement` + `docs`)

Ordered roughly by value/effort. `Status` is ours to maintain (TODO / WIP / PR #… / Done).

| Pri | Issue | Title | Effort | In scope? | Status |
|-----|-------|-------|--------|-----------|--------|
| ✅ | [#754](https://github.com/nginx-proxy/acme-companion/issues/754) | Docker API over TLS-protected socket | medium | ✅ | **Done — merged via PR #1265** |
| ✅ | [#918](https://github.com/nginx-proxy/acme-companion/issues/918) | acme.sh log hardcoded to `/dev/null` even with `DEBUG=1` | low | ✅ | **Done — merged via PR #1266** |
| ✅ | [#710](https://github.com/nginx-proxy/acme-companion/issues/710) | Standalone cert config lacks `listen [::]:80;` (IPv6) | low | ✅ (maintainer-endorsed) | **Done — merged via PR #1267** |
| ✅ | [#1191](https://github.com/nginx-proxy/acme-companion/issues/1191) | Make `RENEW_PRIVATE_KEYS` per-container | low | ✅ | **Done — merged via PR #1268** |
| ✅ | [#881](https://github.com/nginx-proxy/acme-companion/issues/881) | `cert_status` falsely reports valid certs as EXPIRED | low–med | ✅ | **Done — merged via PR #1270** |
| ✅ | [#664](https://github.com/nginx-proxy/acme-companion/issues/664) | Add self-contained Docker Compose **v3** example | low | ✅ (`docs`) | **Done — merged via PR #1272** |
| 🔄 | [#709](https://github.com/nginx-proxy/acme-companion/issues/709) | Add Docker `HEALTHCHECK` | medium | ✅ (maintainer-endorsed) | **PR #1273 (open, awaiting review)** |
| 🔄 | [#956](https://github.com/nginx-proxy/acme-companion/issues/956) | Keep valid cert symlinks on restart (don't cleanup before docker-gen data) | medium | ✅ (maintainer-liked) | **PR #1274 (open, awaiting review)** |
| 🔄 | [#1006](https://github.com/nginx-proxy/acme-companion/issues/1006) | Multi-replica nginx: loop reload over all matching container IDs | medium | ✅ (single-node) | **PR #1275 (open, awaiting review)** |
| 9 | [#997](https://github.com/nginx-proxy/acme-companion/issues/997) | Honor nginx-proxy `CERT_NAME` for generated cert filenames | medium | ✅ | TODO |
| — | [#1062](https://github.com/nginx-proxy/acme-companion/issues/1062) | LE-signed default certificate | medium | ⚠️ mostly nginx-proxy side (`DEFAULT_CERT_NAME`) | TODO |

**Suggested starters:** with #754/#918/#710/#1191/#881/#664 merged and **#709** (PR #1273) + **#956** (PR #1274) + **#1006** (PR #1275) in review, the next clean in-scope pick is **#997** (honor `CERT_NAME`).

---

## Detailed findings

> Grouped by classification. Each entry records the verified verdict and the recommended action.

### `implement` — needs a code change

#### #754 — function docker_api : Using protected socket
- **Created:** 2021-02 (~5.3y) · **Effort:** medium (handled)
- **Claim:** `docker_api` only supports the plain UNIX socket; add TLS-protected `tcp://…:2376` support (`DOCKER_CERT_PATH`, client cert/key/ca). Secondary ask: configurable acme.sh path / non-root.
- **Verification:** **true.** `docker_api` had no TLS branch. **Already addressed by our PR #1265** (adds `DOCKER_TLS_VERIFY`/`DOCKER_CERT_PATH` + `--cert/--key/--cacert`). The non-root/`ACMESH_PATH` sub-request is out of #1265's scope.
- **Action:** **Done — merged via [PR #1265](https://github.com/nginx-proxy/acme-companion/pull/1265) (2026-06-18); issue closed.** The non-root/`ACMESH_PATH` sub-request remains a separate decision.

#### #918 — acme.sh logs still sent to /dev/null in DEBUG mode
- **Created:** 2022-01 (~4.4y) · **Effort:** low
- **Claim:** Even with `DEBUG=1`, acme.sh's own log is hardcoded `--log /dev/null`, hiding detail needed for DNS-01 debugging.
- **Verification:** **true.** `app/letsencrypt_service:154` unconditionally adds `--log /dev/null`; `DEBUG=1` only adds `--debug 2` (line 156). acme.sh's log is never viewable.
- **Action:** **Done — merged via [PR #1266](https://github.com/nginx-proxy/acme-companion/pull/1266) (2026-06-17); issue closed.** Under `DEBUG=1`, acme.sh `--log` now routes to `/dev/stderr` (visible in `docker logs`), keeping `/dev/null` as the non-debug default. (`/dev/stderr` rather than `/dev/stdout`: acme.sh captures helper stdout via command substitution, so logging to stdout could corrupt the ACME exchange.)

#### #710 — Standalone certs fail with IPv6 enabled
- **Created:** 2020-11 (~5.5y) · **Effort:** low
- **Claim:** `standalone-cert-<domain>.conf` only emits `listen 80;`, no `listen [::]:80;`, breaking HTTP-01 on IPv6-only/dual-stack hosts.
- **Verification:** **true.** `app/functions.sh:138-152` (`add_standalone_configuration`) emits only `listen 80;`; no IPv6 plumbing anywhere in the standalone path.
- **Action:** **Done — merged via [PR #1267](https://github.com/nginx-proxy/acme-companion/pull/1267) (2026-06-18); issue closed.** `add_standalone_configuration` now appends `listen [::]:80;` when `ENABLE_IPV6` is truthy (env-var option, matching nginx-proxy's variable name; opt-in so non-IPv6 hosts are unaffected). Maintainer (buchdag) endorsed exactly this in-thread.

#### #1191 — Why is RENEW_PRIVATE_KEYS globally set only?
- **Created:** 2025-04 (~14mo) · **Effort:** low
- **Claim:** Make `RENEW_PRIVATE_KEYS` per-container (use case: stable key for a DANE/TLSA mail cert while rotating elsewhere).
- **Verification:** **true.** Read once globally at `app/letsencrypt_service:10`, applied for all domains at line 462. No `RENEW_PRIVATE_KEYS_${cid}` lookup, unlike neighboring per-container options (`ACME_${cid}_CA_URI` etc.).
- **Action:** **Done — merged via [PR #1268](https://github.com/nginx-proxy/acme-companion/pull/1268) (2026-06-19); issue closed.** Adds `LETSENCRYPT_${cid}_RENEW_PRIVATE_KEYS` (set `LETSENCRYPT_RENEW_PRIVATE_KEYS` on the proxied container), falling back to the global, following the existing per-container nameref pattern + data template + integration test.

#### #881 — cert_status falsely reports valid certs as EXPIRED
- **Created:** 2021-10 (~4.7y) · **Effort:** low–medium
- **Claim:** `/app/cert_status` shows every `fullchain.pem` as EXPIRED though certs are valid (DST Root CA X3 expiry, Sep 2021).
- **Verification:** **true (cosmetic, `cert_status` only).** `openssl verify -untrusted chain.pem fullchain.pem` fails when a cross-signed trust anchor expires → the old code printed `EXPIRED` despite a valid leaf. Cert/renewal logic itself is unaffected.
- **Action:** **Done — merged via [PR #1270](https://github.com/nginx-proxy/acme-companion/pull/1270) (2026-06-19); issue closed.** When chain verification fails, `cert_status` now runs `openssl x509 -checkend 0` on the leaf: a still-valid leaf prints `OK (chain verification failed)` instead of `EXPIRED`; only a genuinely expired leaf still reports `EXPIRED` (`app/cert_status:47-60`).

#### #709 — Support Docker health checks
- **Created:** 2020-11 (~5.6y) · **Effort:** medium
- **Claim:** Provide a Docker `HEALTHCHECK` so `compose up --wait` can gate on readiness.
- **Verification:** **true.** No `HEALTHCHECK` in `Dockerfile`, no health script anywhere. `app/start.sh` backgrounds `letsencrypt_service` + `docker-gen` but its wait loop only monitors the docker-gen PID — if `letsencrypt_service` dies the container stays "up" while renewal silently stops.
- **Action:** **PR [#1273](https://github.com/nginx-proxy/acme-companion/pull/1273) opened (awaiting review).** `start.sh` records both PIDs under `/var/run`; new `app/healthcheck.sh` checks both with `kill -0`; `Dockerfile` adds `HEALTHCHECK` (interval 30s/timeout 5s/start-period 30s/retries 3). PID files (not `pgrep`) avoid matching docker-gen, whose args contain `letsencrypt_service_data`. Adds a `container_health` integration test + `docs/Docker-Compose.md` `--wait`/`service_healthy` note. Verified locally: status flips healthy→unhealthy on `letsencrypt_service` death.

#### #956 — Better starting flow with default certificate
- **Created:** 2022-04 (~4y) · **Effort:** medium
- **Claim:** After recreating the companion, it serves the self-signed default cert while sequentially re-validating each domain, causing transient TLS errors (e.g. Nextcloud sync breaks).
- **Verification:** **true — root cause corrected.** The earlier note ("symlinks are not deleted on restart") was **wrong**. `app/start.sh` launches `letsencrypt_service` (→ immediate `update_certs`) and `docker-gen` in parallel, so the first `update_certs` runs before `/app/letsencrypt_service_data` exists. With no enabled domains, `cleanup_links` (`letsencrypt_service:589`) classifies every persisted `*.crt` symlink as disabled and **deletes them all** (they carry a `.companion` marker). They are then recreated one domain at a time, serving the default cert in between.
- **Action:** **PR [#1274](https://github.com/nginx-proxy/acme-companion/pull/1274) opened (awaiting review).** Minimal root-cause fix: run `cleanup_links` **only once `/app/letsencrypt_service_data` exists**, so the enabled-domain set is complete before any pruning (re-triggered via SIGUSR1 once docker-gen writes the data). An earlier revision gated on *both* data files being absent, but that left a hole (review catch): a **mounted `letsencrypt_user_data` present + `service_data` not yet generated** let cleanup run with only standalone domains enabled and prune the proxied symlinks — gating cleanup on `service_data` alone closes it while the legitimate "all containers removed" cleanup (service_data present, empty) still runs and standalone issuance is not deferred. Deterministic `certs_persistence` test covers both no-user-data and user-data-present cases. The "dir present but symlink missing" migration edge is left out of scope. (The original "relink before validation" idea became unnecessary once symlinks stop being deleted.)

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

### `docs` — documentation only

#### #664 — Doc updates for V3 Docker compose?
- **Created:** 2020-06 (~6y) · **Effort:** low · label `type/docs`
- **Claim:** Docs should include self-contained Compose **v3** examples (current ones are v2 with `volumes_from`, or link out).
- **Verification:** **true.** `docs/Docker-Compose.md` examples used the obsolete top-level `version:` key + `volumes_from` (a compose-file-v2-only feature); self-contained/modern guidance deferred to external links.
- **Action:** **Done — merged via [PR #1272](https://github.com/nginx-proxy/acme-companion/pull/1272) (2026-06-20); issue closed.** Rewrote both compose examples: dropped `version:` (obsolete in the Compose Specification), replaced `volumes_from` with explicit per-container volume mounts, switched container detection to the `label` method (`com.github.nginx-proxy.nginx` / `com.github.nginx-proxy.docker-gen`), mounted the shared `html` volume rw on the companion (HTTP-01 writes), and (review catch) added `certs:ro` to the three-container docker-gen so `nginx.tmpl` detects issued certs at render time. Validated with `docker compose config`. Scope limited to `docs/Docker-Compose.md`; the `docker run --volumes-from` docs are left intact.

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
- **2025-11.** Decisive log `retryafter=86400 … too large (>600), will not retry` is from **acme.sh** (ZeroSSL returned 24h Retry-After while order stayed Pending) — a ZeroSSL/DNS-validation problem, not companion code. → Ask for LE-staging retest + confirm `_acme-challenge` TXT propagation. Likely closes as `invalid`.

#### #1196 — Unneeded renewal makes the service go down
- **2025-05.** Premature renewal not reproduced (acme.sh decides via `--days`; restart doesn't force renew). The ~5-min outage is likely ZeroSSL issuance latency / nginx-proxy reload, not deliberate cert removal. → Ask for `DEBUG=1` logs of the triggering restart + cert timestamps.

#### #1042 — crt/key not created in certs folder
- **2023-07.** No companion logs provided; symptoms match the curl-28/#1084 hang. → Request `DEBUG=1` logs; if `error code: 28`/"Can not init api", close as duplicate of #1084.

### `question` — usage/support, no code change

#### #1000 — Nginx-proxy challenges failing
- **2023-02 · label `kind/failing-authorization`.** LE gets `502`/`500` from the user's domain → backend not serving the challenge / not on the proxy network. Matches `docs/Invalid-authorizations.md`. → Redirect to docs.

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

- With **#754** (PR #1265), **#918** (PR #1266), **#710** (PR #1267), **#1191** (PR #1268), **#881** (PR #1270) and **#664** (PR #1272) merged, and **#709** (PR #1273) + **#956** (PR #1274) + **#1006** (PR #1275) in review, the next clearly actionable, in-scope, low-risk item is **#997** (honor `CERT_NAME`).
- Several `invalid`/`question`/`failing-authorization` issues all reduce to the same root causes documented in `docs/Invalid-authorizations.md` (shared volumes, IPv6/AAAA reachability, app on the proxy network).
- `out-of-scope` here means "not addressable in this repo" (Swarm, host networking, host OS, or an upstream nginx-proxy/acme.sh concern) — several have maintainer statements on record.
- This is a point-in-time snapshot (2026-06-17); verify status before acting, as upstream may have moved.
