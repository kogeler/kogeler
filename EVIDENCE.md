# Selected work

<!-- CVs link to the headings of this page, and a sent CV keeps the address: do not rename or remove a heading. One statement per line, do not rewrap lines. -->

**This page uses the names from my CV. Each section first says what the CV calls the work and what part of it is not public, then shows the public part or public work that follows the same approach.**

These are examples, many of them historical: the latest versions of many systems are in private company repositories.
Some of my own projects are published on purpose as examples of private work, and some are the foundation of solutions I build at work.

I am [kogeler](https://github.com/kogeler) on GitHub. Every pull request, issue and repository linked here is mine unless a line says otherwise.
Covered: systems I built for employers, fixes in widely used open-source software, design work in my own projects. Each statement ends with links to the code, review or issue that shows it.

## At a glance

| Case | Where | Years | Shows |
| --- | --- | --- | --- |
| [CDN and app-store exporters](#cdn-and-app-store-exporters) | Novasama | 2025-2026 | Custom Prometheus exporters, metrics correctness, HA collection |
| [Database backup and migration tool](#database-backup-and-migration-tool) | Novasama | 2025-2026 | Managed PostgreSQL backups, verified restores |
| [Mobile release pipelines and signing](#mobile-release-pipelines-and-signing) | Novasama | 2025-2026 | Android and iOS release CI, signing keys out of repositories, short-lived tokens |
| [Containerized builds and tests](#containerized-builds-and-tests) | Novasama | 2026 | Same builds locally and in CI, one generic workflow, tests in the runtime image |
| [Review and merge gates](#review-and-merge-gates) | Novasama | 2025-2026 | Branch rulesets, required checks, review before merge |
| [Node binary builds](#node-binary-builds) | Novasama | 2024-2026 | Per-release node binaries, CPU-level and hardening build profiles |
| [Self-hosted indexers](#self-hosted-indexers) | Novasama | 2024-2026 | Data pipelines into PostgreSQL, TypeScript, production incident fixes |
| [Public snapshot service](#public-snapshot-service) | Parity | 2023 | Backups of large node databases, object storage behind a CDN, restore clients |
| [Node health exporters](#node-health-exporters) | Parity | 2023-2024 | Custom Prometheus exporters, Kubernetes probes |
| [Public Ansible collection](#public-ansible-collection) | Parity | 2022-2024 | Cloud-agnostic Ansible roles, Molecule CI, releases |
| [NFS client fixes](#nfs-client-fixes) | Upstream | 2026 | C, protocol encoding, session handling |
| [Xpra Wayland and protocol fixes](#xpra-wayland-and-protocol-fixes) | Upstream | 2026 | Python and Cython, Wayland, packaging |
| [HAProxy HTTP/2 demuxer bug](#haproxy-http2-demuxer-bug) | Upstream | 2026 | Root-cause analysis at protocol and packet level |
| [Network edge and firewall](#network-edge-and-firewall) | Own | 2023-2026 | nftables, NAT, TLS routing by SNI and ALPN, hardened Podman services |
| [Remote access for AI agents](#remote-access-for-ai-agents) | Own | 2026 | MCP servers, OpenSSH, hardware keys |
| [Kernel builds and signed releases](#kernel-builds-and-signed-releases) | Own | 2026 | Kernel builds and tests, signing keys in CI, provenance |
| [Self-hosted Kubernetes](#self-hosted-kubernetes) | Own | 2026 | Cluster API on LXD, Terraform, Ansible, Helm |
| [Spec-driven development](#spec-driven-development) | Own | 2026 | Testable specifications, rules for coding agents |

## Employer work

### CDN and app-store exporters

**In my CV: custom exporters for CDN and app-store metrics. The exporters are public; the monitoring platform they feed (storage, alert routing, dashboards) is not.**

Novasama Technologies, 2025-2026 · Python, Prometheus, Cloudflare API, App Store Connect, Google Play

Data that no standard exporter provides: CDN traffic, app-store installs, TURN relay health.

* **Cloudflare exporter.** Cloudflare HTTP analytics for Prometheus as closed 5-minute buckets, with health metrics and example alerts ([design](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md)).
* **Correctness.** A bucket is exported only when its groups sum to independently read totals; sampled responses under load are detected and re-read, so data arrives late, never wrong ([rationale](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md#why-a-bucket-is-trusted)).
* **App-store exporters.** Installs, uninstalls and active devices by country ([App Store Connect](https://github.com/novasamatech/infra-tooling/blob/main/app-store-exporter/README.md), [Google Play](https://github.com/novasamatech/infra-tooling/blob/main/gplay-exporter/README.md)).

<details>
<summary>More: HA collection, tests and cardinality, relay probe</summary>

* **HA collection.** Replicas emit identical series and timestamps for store-side deduplication; each rebuilds its state after a lost or corrupt database ([infra-tooling#34](https://github.com/novasamatech/infra-tooling/pull/34), [storage](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md#storage)).
* **Tests and cardinality.** Offline tests against an in-memory fake of the Cloudflare API that reproduces sampled responses; a live check of exported closed-hour sums against independent totals; per-metric label disabling ([development](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md#development), [label selection](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md#label-selection)).
* **Relay probe.** STUN, TURN allocation and relay-only WebRTC probes of a CoTURN server, with alert rules ([infra-tooling#24](https://github.com/novasamatech/infra-tooling/pull/24)).

</details>

### Database backup and migration tool

**In my CV: backups of the platform's state, from database dumps to persistent volumes. Shown here: the dump and restore tool for one group of databases. The scheduled backups and the code that runs the tool are not public.**

Novasama Technologies, 2025-2026 · Python, PostgreSQL, S3, Podman

Managed PostgreSQL gives the indexer databases no superuser or WAL access, and a standard logical restore failed after many hours on very large index builds.

* **Backup and migration tool.** `sqdb`, a containerized tool: per-database backups and moves between managed PostgreSQL instances and major versions ([infra-tooling#30](https://github.com/novasamatech/infra-tooling/pull/30)).
* **Streaming to object storage.** Dumps stream to S3 as multipart uploads, without local disk; a failed upload never leaves a truncated object ([infra-tooling#30](https://github.com/novasamatech/infra-tooling/pull/30)).
* **Restore for index-heavy databases.** Four-phase restore that creates indexes on empty tables before loading data, avoiding the index builds that failed a standard restore ([design rationale](https://github.com/novasamatech/infra-tooling/blob/main/subquery-db/README.md)).

<details>
<summary>More: verified restores, rehearsal</summary>

* **Verified restores.** DDL probe before a restore; catalog and row verification after it, also for dumps taken while the source keeps writing ([infra-tooling#33](https://github.com/novasamatech/infra-tooling/pull/33)).
* **Rehearsal.** Fixed a failure found in a real restore, added an end-to-end regression and rehearsed against a 37-database configuration ([infra-tooling#32](https://github.com/novasamatech/infra-tooling/pull/32)).

</details>

### Mobile release pipelines and signing

**In my CV: the secured release path, where app signing keys exist only in CI release jobs. Shown here: the release pipelines and signing of the mobile apps. The backend pipelines, with image and dependency scanning and GitOps deployment, are not public.**

Novasama Technologies, 2025-2026 · GitHub Actions, Fastlane, Google Play, TestFlight, Firebase, cloud secret manager

CI builds, signs and uploads the releases of the Nova Wallet mobile apps and Polkadot Vault.

* **Signing keys out of repository secrets.** Signing keystores and store credentials moved to a cloud secret manager, decoded only inside CI build and upload jobs ([nova-wallet-android#2276](https://github.com/novasamatech/nova-wallet-android/pull/2276)).
* **Android release pipeline.** Rebuilt Nova Wallet Android CI: signed bundles, Google Play upload with track control, parallel test and build jobs in a reusable workflow ([nova-wallet-android#2276](https://github.com/novasamatech/nova-wallet-android/pull/2276)).
* **iOS release pipeline.** Nova Wallet iOS CI on Fastlane lanes: build, tests, Firebase, TestFlight, `match` code signing ([nova-wallet-ios#1781](https://github.com/novasamatech/nova-wallet-ios/pull/1781)).

<details>
<summary>More: short-lived tokens, Polkadot Vault distribution, release notifications, key isolation in my own projects</summary>

* **Short-lived tokens.** In the workflow that publishes design tokens to the iOS and Android repositories: GitHub App tokens scoped to one target repository each instead of a long-lived personal access token, an in-repo action instead of a third-party one ([paritytech/polkadot-app-design-system#6](https://github.com/paritytech/polkadot-app-design-system/pull/6)).
* **Polkadot Vault distribution.** Firebase distribution of the Android and iOS builds ([parity-signer#2478](https://github.com/novasamatech/parity-signer/pull/2478)).
* **Release notifications.** Shared actions collect a release's pull requests and send notifications with download links ([github-actions#7](https://github.com/novasamatech/github-actions/pull/7), [github-actions#3](https://github.com/novasamatech/github-actions/pull/3)).
* **Key isolation in my own projects.** A signing key reachable by one CI job only, and releases without a long-lived token: [Kernel builds and signed releases](#kernel-builds-and-signed-releases).

</details>

### Containerized builds and tests

**In my CV: containerized builds and tests that run the same locally and in CI. Shown here: the public repositories of infrastructure tools and indexers. The backend services, built and tested the same way, are not public.**

Novasama Technologies, 2026 · Podman, Docker, Make, GitHub Actions

The infrastructure tools share one repository of independent projects; the indexers have repositories of their own.

* **Same builds locally and in CI.** In the infrastructure tools repository, builds and tests run through the same containerized `make` targets: Podman locally, Docker in CI ([repository rules](https://github.com/novasamatech/infra-tooling/blob/main/AGENTS.md#continuous-integration)).
* **Generic workflow.** One workflow finds the projects a change touches, runs their checks and publishes images only after the checks pass ([infra-tooling#30](https://github.com/novasamatech/infra-tooling/pull/30), [workflow](https://github.com/novasamatech/infra-tooling/blob/main/.github/workflows/ci.yml)).
* **Tests in the runtime image.** Indexer offline tests run inside the final runtime image during the production image build ([subquery-accounts#97](https://github.com/novasamatech/subquery-accounts/pull/97)).
* **CI reproduced locally.** A rootless Podman workflow reproduces an indexer's CI on a developer machine ([subquery-accounts#92](https://github.com/novasamatech/subquery-accounts/pull/92)).

### Review and merge gates

**In my CV: engineering practices of the product teams, enforced by the Git workflow and CI/CD. Shown here: that enforcement in four public repositories. The written specifications that precede implementation and the private repositories are not public.**

Novasama Technologies, 2025-2026 · GitHub rulesets, required status checks, pull request review

The repository enforces review, not an agreement: a change reaches the default branch only through a reviewed pull request.

* **Merge gates.** Branch rulesets require an approval and resolved review threads before a merge and forbid force-pushes to the default branch (rulesets: [infra-tooling](https://github.com/novasamatech/infra-tooling/rules/4294674), [github-actions](https://github.com/novasamatech/github-actions/rules/10951966), [subquery-accounts](https://github.com/novasamatech/subquery-accounts/rules/6633907), [nova-utils](https://github.com/novasamatech/nova-utils/rules/27362926)).
* **Required checks.** Integration tests became a required check before a merge in the wallet configuration repository ([nova-utils#4474](https://github.com/novasamatech/nova-utils/pull/4474)).
* **The same gates for my own changes.** A teammate's review of one of my pull requests raised ten findings; I answered each with a change in the same pull request before the approval ([infra-tooling#30](https://github.com/novasamatech/infra-tooling/pull/30)).

### Node binary builds

**In my CV: node client releases rolled out across validators and RPC nodes. Shown here: the pipeline that builds those binaries. The rollout is not public.**

Novasama Technologies, 2024-2026 · GitHub Actions, Rust and Clang toolchains, container images

Novasama's validators and RPC nodes run these binaries. The pipeline began in my own repository; the company's fork of it is now the official source.

* **Optimized node binaries.** Compiles the Polkadot SDK node binaries of each release from a matrix of build profiles, inside a versioned build image, for the `x86-64-v3` CPU level with hardened compiler flags; publishes GitHub releases and container images ([build workflow](https://github.com/novasamatech/substrate-cross-compiler-env/blob/main/.github/workflows/polkadot.yml), [build profiles](https://github.com/novasamatech/substrate-cross-compiler-env/blob/main/config.yml), [releases](https://github.com/novasamatech/substrate-cross-compiler-env/releases)).

### Self-hosted indexers

**In my CV: over 100 SubQuery indexers. Shown here: their code and network configuration. Their Kubernetes deployment, scaling for full reindexing and PostgreSQL tuning are not public.**

Novasama Technologies, 2024-2026 · TypeScript, SubQuery, PostgreSQL, GraphQL, Podman

Nova Wallet reads transfer history, staking, governance, multisig and proxy data from SubQuery indexers: TypeScript projects that stream chain data into PostgreSQL and serve it through GraphQL.

* **Move to self-hosted indexers.** Moved the wallet from a hosted SubQuery service to company-run indexers: configuration switched in development first, then in production ([nova-utils#3791](https://github.com/novasamatech/nova-utils/pull/3791)).
* **Scale.** 145 network manifests in four indexer repositories; 51 of the 98 chains in the production wallet configuration read from company-run indexers ([production chain configuration](https://github.com/novasamatech/nova-utils/blob/master/chains/v23/chains.json)).
* **Asset Hub decoding incident.** A new extrinsic format crashed production on Polkadot Asset Hub; fixed with a metadata-driven decoder ported to four indexers and three Asset Hubs, verified by replaying real blocks in the SubQuery runtime ([subquery-accounts#97](https://github.com/novasamatech/subquery-accounts/pull/97), [subquery-accounts#99](https://github.com/novasamatech/subquery-accounts/pull/99)).

<details>
<summary>More: stalled history indexer, Polkadot.js v16 migration, other indexer work</summary>

* **Stalled history indexer.** A 5 KB post-quantum authorization proof in an indexed address column stalled the Bittensor history indexer; added validation to every address writer and a regression test on the real block ([subquery-history#345](https://github.com/novasamatech/subquery-history/pull/345)).
* **Polkadot.js v16 migration.** Indexers moved to `@polkadot/*` v16, with fixes to runtime types, module formats and Asset Hub type boundaries ([subquery-staking#99](https://github.com/novasamatech/subquery-staking/pull/99), [subquery-accounts#87](https://github.com/novasamatech/subquery-accounts/pull/87)).
* **More indexer work.** Cross-network decoding, call traversal and pure-proxy fixes: [Novasama SubQuery work](UPSTREAM.md#novasama-subquery-work).

</details>

### Public snapshot service

**In my CV: a public snapshot service, called a public backup service in my general CV. Shown here: its first generation. The Kubernetes pipelines that replaced it, which I also built and the CV describes, are not public.**

Parity Technologies, 2023 · Ansible, Helm, rclone, Cloudflare R2, Prometheus

I designed and built [snapshots.polkadot.io](https://snapshots.polkadot.io/): Parity's database snapshots of Polkadot, Kusama, test networks and system parachains, so a node starts without syncing the chain from its first block.

* **Root cause of broken downloads.** Downloads of 40+ GB database files failed because the CDN edge cached unrequested `206 Partial Content` replies from R2, and because of three `rclone` defaults; restore code fixed in the Ansible role and the node Helm chart ([ansible-polkadot#42](https://github.com/paritytech/ansible-polkadot/pull/42), [helm-charts#283](https://github.com/paritytech/helm-charts/pull/283)).
* **Snapshot producer.** `node_backup` role: waits for sync, stops the node for a consistent copy, uploads only changed files to Cloudflare R2 or GCS with `rclone`, checks the remote file list against the local one, publishes size, last-block and version metadata ([backup script](https://github.com/paritytech/ansible-polkadot/blob/main/roles/node_backup/templates/single-backup.sh.j2), [ansible-polkadot#48](https://github.com/paritytech/ansible-polkadot/pull/48)).
* **Backup monitoring.** Prometheus exporter in the role for the time, size and last block of each backup ([exporter](https://github.com/paritytech/ansible-polkadot/blob/main/roles/node_backup/files/exporter.py)).

<details>
<summary>More: restore client, design constraints</summary>

* **Restore client.** HTTP restore in the node role: reads snapshot metadata, checks free disk space, downloads files in parallel ([ansible-polkadot#36](https://github.com/paritytech/ansible-polkadot/pull/36)).
* **Design constraints.** Explained to the database maintainers why snapshots ship as per-file resumable downloads: 160-200 GB table files, retries on huge files, CDN object size limits ([parity-db#219](https://github.com/paritytech/parity-db/issues/219)).

</details>

### Node health exporters

**In my CV: custom exporters for node and validator state and for the public RPC API. The exporters are public; the dashboards and alerts built on them and the rest of the monitoring stack are not.**

Parity Technologies, 2023-2024 · Python, Prometheus, Kubernetes probes, Ansible

Signals that a node's own metrics do not give: whether an endpoint answers, keeps up with the chain and holds the right session keys.

* **Health exporter for RPC endpoints.** Extended `ws-health-exporter` from a teammate's single-endpoint check into a service that watches several WebSocket endpoints and reports a node unhealthy on low peer count, lost sync or block-height drift ([scripts#533](https://github.com/paritytech/scripts/pull/533), [exporter](https://github.com/paritytech/scripts/blob/master/dockerfiles/ws-health-exporter/exporter.py)).
* **Block-rate check.** Minimum block-production-rate check; logs name the failed check ([scripts#592](https://github.com/paritytech/scripts/pull/592)).
* **Liveness probe.** The exporter as a sidecar liveness probe in the node Helm chart restarts RPC nodes that stop accepting WebSocket connections ([helm-charts#203](https://github.com/paritytech/helm-charts/pull/203)).
* **Validator state exporter.** `state_exporter` finds node processes on a host, reports their CPU, memory and threads, and rebuilds complete validator and collator session keys from the keystore ([exporter](https://github.com/paritytech/ansible-polkadot/blob/main/roles/state_exporter/files/exporter.py), [ansible-polkadot#48](https://github.com/paritytech/ansible-polkadot/pull/48)).
* **Session-key formats.** Added BEEFY-era keys and keys injected at different times ([ansible-polkadot#65](https://github.com/paritytech/ansible-polkadot/pull/65)).

### Public Ansible collection

**In my CV: the team's Ansible roles, rewritten with Molecule tests. Shown here: the roles that could be published. Most of the roles and the Terraform modules are not public.**

Parity Technologies, 2022-2024 · Ansible, Molecule, GitHub Actions, Ansible Galaxy, Nginx

`paritytech.chain`, Parity's public Ansible collection for running nodes: more than 7,000 downloads on [Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/paritytech/chain/), still developed by Parity.

* **Molecule CI.** Multi-role Molecule tests in GitHub Actions on LXD, because Docker could not run systemd when host and guest OS versions differ; moved to the Docker driver when the LXD driver was archived ([ansible-polkadot#32](https://github.com/paritytech/ansible-polkadot/pull/32), [ansible-polkadot#62](https://github.com/paritytech/ansible-polkadot/pull/62)).
* **RPC front end.** Nginx role for RPC nodes: routes HTTP and WebSocket JSON-RPC to the right node port, manages Let's Encrypt or custom certificates, applies request rate limits; its Molecule tests check HTTPS and WSS endpoints against a local ACME server ([ansible-polkadot#62](https://github.com/paritytech/ansible-polkadot/pull/62)).
* **Role refactoring.** Production node role refactored for community use: users, data paths, task layout, tags, variables, tests, documentation ([ansible-polkadot#2](https://github.com/paritytech/ansible-polkadot/pull/2)).

<details>
<summary>More: releases, lint and locked dependencies, cloud-agnostic restore</summary>

* **Releases.** Reusable workflows: per-role Molecule jobs, a version check, automatic publication to Ansible Galaxy ([ansible-polkadot#47](https://github.com/paritytech/ansible-polkadot/pull/47)).
* **Lint and locked dependencies.** All `ansible-lint` findings fixed across the collection; Python dependencies of the test environment locked ([ansible-polkadot#64](https://github.com/paritytech/ansible-polkadot/pull/64), [ansible-polkadot#68](https://github.com/paritytech/ansible-polkadot/pull/68)).
* **Cloud-agnostic restore.** Plain HTTP and `rclone` replaced GCP-specific restore code, so restores depend on no single provider's tools ([ansible-polkadot#63](https://github.com/paritytech/ansible-polkadot/pull/63)).

</details>

## Upstream fixes

**In my CV: software development. Most of the code I write for employers is not public. Shown here: my code merged into open-source projects that I do not maintain.**

C in an NFS client library, Python and Cython in a remote display server, a protocol-level bug in HAProxy.
The maintainers merged or applied each fix; the HAProxy maintainers confirmed the bug I reported and fixed it.

### NFS client fixes

Upstream, 2026 · C, NFSv4.2, FUSE

libnfs is a user-space NFS client library; fuse-nfs mounts NFS shares through it. Each fix has a regression test and was validated with sanitizers and real NFS workloads.

* **Payload alignment.** Fixed the alignment of allocated RPC payloads ([libnfs#608](https://github.com/sahlberg/libnfs/pull/608)).
* **Empty fields.** Fixed the encoding of zero-length byte payloads, which passed NULL to `memcpy` ([libnfs#610](https://github.com/sahlberg/libnfs/pull/610)).
* **Safe retries.** Asynchronous retries of NFSv4.2 `NFS4ERR_DELAY` replies for requests that are safe to replay; applied by the maintainer with minor fixes ([libnfs#609](https://github.com/sahlberg/libnfs/pull/609)).
* **Stale filehandles.** Stale NFSv4 filehandles map to `ESTALE`, so filesystem adapters can revalidate them ([libnfs#614](https://github.com/sahlberg/libnfs/pull/614)).
* **Idle sessions.** A background service loop in fuse-nfs keeps idle NFSv4.2 sessions alive ([fuse-nfs#53](https://github.com/sahlberg/fuse-nfs/pull/53)).

### Xpra Wayland and protocol fixes

Upstream, 2026 · Python, Cython, Wayland, VA-API, Debian packaging

Xpra is the remote display server that [Elsewindow](PROJECTS.md#elsewindow) builds on. 14 of my pull requests are merged: [list](https://github.com/Xpra-org/xpra/pulls?q=is%3Apr+author%3Akogeler+is%3Amerged).

* **Wayland frame callbacks.** Fixed frame callbacks for commits with empty damage ([xpra#5002](https://github.com/Xpra-org/xpra/pull/5002)).
* **Wayland window lifecycle.** Stopped refreshing a window destroyed while it is being mapped, and published the opaque region of popups ([xpra#5130](https://github.com/Xpra-org/xpra/pull/5130), [xpra#5138](https://github.com/Xpra-org/xpra/pull/5138)).
* **Hardware video decoding.** Registered the native libva video decoder ([xpra#5005](https://github.com/Xpra-org/xpra/pull/5005)).
* **Protocol errors.** Applied the packet error policy to short packets ([xpra#5155](https://github.com/Xpra-org/xpra/pull/5155)).
* **Packaging.** Fixed Debian upgrades to the split Wayland package ([xpra#5013](https://github.com/Xpra-org/xpra/pull/5013)).

### HAProxy HTTP/2 demuxer bug

**In my CV: packet-level tracing, among the networking skills. Shown here: one investigation, from my own infrastructure.**

Upstream, 2026 · HTTP/2, TCP, HAProxy traces, packet captures

A production HAProxy reset connections intermittently, killing every multiplexed stream on them.

* **Root cause.** HAProxy's HTTP/2 demuxer: under network backpressure it left the padding of a DATA frame in its demux buffer, read it as the next frame header and sent `GOAWAY PROTOCOL_ERROR` ([haproxy#3354](https://github.com/haproxy/haproxy/issues/3354)).
* **Method.** Matched application failures with HAProxy traces, buffer offsets, RTT, congestion and flow-control state across several release branches; this ruled out the backend and showed that tuning only changed the failure rate ([capture and analysis scripts](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/doc/scripts/README.md)).
* **Outcome.** The maintainers confirmed the bug, fixed it and backported the fix ([upstream fix](https://github.com/haproxy/haproxy/commit/faf3e9ac3a5df7258b0abbc06b0e0378617a18e5)).

## Own projects

### Network edge and firewall

**In my CV: firewalling, among the networking skills. Shown here: that skill in my own Ansible collection. Employer firewalls are not public.**

Own project, 2023-2026 · nftables, iptables, HAProxy, sing-box, NaiveProxy, Podman, Molecule

Three roles from my `mini-pig` Ansible collection: a host firewall, and two that deploy censorship-resistant proxies behind a shared TLS port.

* **Firewall that leaves foreign rules alone.** The role owns scoped iptables chains or atomic nftables tables and never flushes the rules of Kubernetes, Docker, libvirt or Podman ([firewall role](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/iptables/README.md)).
* **Randomized SNAT pool.** A SNAT pool over several external addresses in its own nftables chain, at a priority evaluated before kube-proxy and the CNI ([SNAT pool](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/iptables/README.md#randomized-snat-pool)).
* **Firewall tests.** Molecule scenarios: migration from a legacy layout, the move from the iptables backend to nftables, drift recovery and, at packet level, the spread of connections across the SNAT pool ([scenarios](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/iptables/README.md#molecule)).

<details>
<summary>More: SNI and ALPN routing, proxy behind a decoy, hardened Podman services, MTProxy topologies, client addresses behind an SNI router</summary>

* **SNI and ALPN routing.** Proxy edge role: HAProxy inspects SNI and ALPN on one public port; one name passes through to a sing-box AnyTLS server that terminates its own TLS, ACME TLS-ALPN-01 challenges go to the certificate client, TLS terminates for the rest ([architecture](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/README.md#architecture)).
* **Authenticated proxy behind a decoy.** After TLS termination, authenticated proxy requests reach NaiveProxy and all others a decoy web site, so the endpoint looks like an ordinary HTTPS server ([HAProxy contract](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/doc/contracts/haproxy.md)).
* **Hardened Podman services.** The proxy services run as Podman containers under systemd with all capabilities dropped and `no-new-privileges` set ([runtime contract](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/doc/contracts/runtime.md)).
* **Two MTProxy topologies.** Telemt role: direct MTProxy with optional Fake TLS, or an HTTPS variant behind HAProxy serving several domains from one deployment ([traffic path](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/telemt/README.md#traffic-path), [mini-pig-ansible-collection#13](https://github.com/kogeler/mini-pig-ansible-collection/pull/13)).
* **Address handling behind an SNI router.** The role's documentation works out what an SNI TCP router does to client addresses, `X-Forwarded-For` and per-address limits ([client address behind an SNI TCP router](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/telemt/README.md#client-address-behind-an-sni-tcp-router)).

</details>

### Remote access for AI agents

**In my CV: a custom SSH MCP server. This is that server: my own open-source project, which I use at work on non-production hosts.**

Own projects, 2026 · Python, MCP, OpenSSH, FIDO2 keys, rsync, Podman

Remote SSH MCP gives coding agents structured tools for remote Linux hosts instead of a raw terminal. Its transport is `ssh-wrapper`, a typed library published on PyPI.

* **Bounded tools.** 13 tools for sessions, bounded commands, inspection and file transfers, each marked read-only or state-changing for the client's approval policy ([tools](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/tools.md)).
* **Hardware keys.** One explicit OpenSSH control connection per session: a FIDO2 key asks for its PIN and touch once per session, and a lost connection is never reopened silently ([security model](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/security.md), [FIDO acceptance test](https://github.com/kogeler/ssh-wrapper/blob/main/tests_acceptance/test_fido.py)).
* **Controlled sudo.** `sudo` never accepts a password and never relies on a cached authentication ([security model](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/security.md)).

<details>
<summary>More: verified transfers, owned remote processes, live tests</summary>

* **Verified transfers.** Large files move through resumable, SHA-256-verified rsync transfers outside the model context ([architecture](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/architecture.md)).
* **Owned remote processes.** `ssh-wrapper` ties a remote process group to its owner with a heartbeat and cleans it up when the owner disappears ([security contract](https://github.com/kogeler/ssh-wrapper/blob/main/docs/contracts/security.md)).
* **Live tests.** Both projects are tested against real OpenSSH in Podman, including hardware-backed keys ([live acceptance](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/development.md#live-acceptance)).

</details>

### Kernel builds and signed releases

**In my CV: fast kernel updates on bare-metal hosts. Shown here: my own project that builds, tests and publishes current kernels. Employer hosts and their configuration are not public.**

Own projects, 2026 · Debian, Clang with ThinLTO, KVM, DKMS, GPG, APT, PyPI Trusted Publishing, GitHub Actions

Debian Kernel Current rebuilds the newest Debian Sid kernel for Debian 13 and publishes it through a signed APT repository. My Python packages are released through PyPI.

* **Fast kernel updates.** A scheduled pipeline checks Debian for a new kernel four times a day and rebuilds it with Clang and ThinLTO for `x86-64-v2` and `x86-64-v3` ([Debian Kernel Current](https://github.com/kogeler/dkc-linux#readme)).
* **Release gates.** Published only after package installation, KVM boot, kernel selftests, DKMS modules, removal and fallback to the stock Debian kernel all pass ([kernel testing](https://github.com/kogeler/dkc-linux/blob/main/docs/KERNEL_TESTING.md)).
* **Signing key isolation.** The archive signing subkey lives in a GitHub Environment restricted to the `main` branch; only the signing job references it, and build and test jobs receive no production secret ([threat model](https://github.com/kogeler/dkc-linux/blob/main/docs/SECURITY.md#reaching-a-signing-key-or-a-storage-credential), [signing Environment](https://github.com/kogeler/dkc-linux/blob/main/docs/KEYS.md#github-environment)).

<details>
<summary>More: trusted publishing, pinned actions and scanning</summary>

* **Trusted publishing.** The Python packages publish CI-built artifacts through PyPI Trusted Publishing with attestations; no long-lived upload token exists ([CI and release contract](https://github.com/kogeler/ssh-wrapper/blob/main/docs/contracts/ci_releases.md)).
* **Pinned actions and scanning.** Every GitHub Action pinned by commit SHA; dependency review and CodeQL on pull requests ([CI workflow](https://github.com/kogeler/ssh-wrapper/blob/main/.github/workflows/ci.yml)).

</details>

### Self-hosted Kubernetes

**In my CV: self-hosted Kubernetes in the company platform. That platform is not public. Shown here: `k8s-lab`, which I built on purpose as a public example.**

Own project, 2026 · Cluster API, LXD, k3s, Terraform, Ansible, Helm, Calico, MetalLB

`k8s-lab` builds a self-hosted Cluster API platform on one Debian or Ubuntu host.

* **Unprivileged nodes.** Kubernetes nodes run in unprivileged LXC containers ([decision record](https://github.com/kogeler/k8s-lab/blob/main/doc/decisions/0002-unprivileged-lxc-nodes.md)).
* **Self-managed management cluster.** A temporary k3s cluster installs Cluster API and hands control to a management cluster that then manages itself ([decision record](https://github.com/kogeler/k8s-lab/blob/main/doc/decisions/0004-bootstrap-cluster-and-mandatory-pivot.md)).
* **Declarative clusters.** Terraform declares the workload clusters, Ansible prepares the host, Helm deploys Cluster API resources, Calico and MetalLB ([architecture](https://github.com/kogeler/k8s-lab/blob/main/doc/architecture.md)).
* **Tests.** Molecule scenarios and an end-to-end workflow in GitHub Actions run the whole flow ([test strategy](https://github.com/kogeler/k8s-lab/blob/main/doc/decisions/0016-test-strategy.md), [end-to-end workflow](https://github.com/kogeler/k8s-lab/blob/main/.github/workflows/e2e.yml)).
* **Operations.** Runbooks cover upgrades, scaling, certificates, backup and restore ([runbooks](https://github.com/kogeler/k8s-lab/blob/main/doc/runbooks/README.md), [backup and restore](https://github.com/kogeler/k8s-lab/blob/main/doc/runbooks/backup-restore.md)).

### Spec-driven development

**In my CV: the company's spec-driven development (SDD) framework. The framework itself, with its project templates and rules for agents, is not public. Shown here: public work that follows the same approach, in my own projects and in two public company repositories.**

Own projects, 2026 · Behavior contracts, decision records, AGENTS.md, pytest

I specify behavior as testable contracts and give coding agents fixed rules, so that an agent can change a project without breaking what it promises.

* **Contract catalogs.** Numbered `MUST`-level assertions with permanent IDs, each linked to the automated checks that prove it; 208 assertions in three repositories ([joplin-md-sync](https://github.com/kogeler/joplin-md-sync/blob/main/docs/contracts/README.md), [ssh-wrapper](https://github.com/kogeler/ssh-wrapper/blob/main/docs/contracts/README.md), [fmi-hass-custom](https://github.com/kogeler/fmi-hass-custom/blob/master/docs/contracts/README.md)).
* **Enforced traceability.** A test fails when an assertion has a duplicate ID or links to evidence that does not exist ([traceability test](https://github.com/kogeler/joplin-md-sync/blob/main/tests/unit/test_documentation_contracts.py)).
* **Agent rules in employer repositories.** Public Novasama repositories carry my agent rules: one generic CI and container-only tasks in the infrastructure tools, mandatory verification in the real runtime in an indexer ([infra-tooling AGENTS.md](https://github.com/novasamatech/infra-tooling/blob/main/AGENTS.md), [subquery-accounts AGENTS.md](https://github.com/novasamatech/subquery-accounts/blob/main/AGENTS.md)).

<details>
<summary>More: decision records, rules for coding agents</summary>

* **Decision records.** `k8s-lab` records its design in 16 architecture decisions ([decisions](https://github.com/kogeler/k8s-lab/blob/main/doc/decisions/README.md)).
* **Rules for coding agents.** An `AGENTS.md` entry page sets a working contract, documentation routes and a change and verification flow ([dkc-linux AGENTS.md](https://github.com/kogeler/dkc-linux/blob/main/AGENTS.md)).

</details>
