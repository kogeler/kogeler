# Upstream work and public contributions

This page collects public issues and merged changes in open-source projects,
plus a selection from public employer repositories. They show how I debug
failures, build operational tooling, and maintain distributed systems.

## Debugging and community work

### HAProxy HTTP/2 demuxer failure

Tracked a production HTTP/2 failure to a demux-buffer transition under network
backpressure. HAProxy misread a valid frame, sent `GOAWAY PROTOCOL_ERROR`, and
closed every multiplexed stream on the connection.

Matched application failures with HAProxy traces, internal buffer offsets, RTT,
congestion, and flow-control state across several release branches. The work
ruled out the backend and showed that configuration tuning only changed the
likelihood of failure. HAProxy maintainers confirmed an HTTP/2 subsystem bug and
closed the report as fixed.

* [haproxy/haproxy#3354 - H2 demuxer race after `demux buffer full`](https://github.com/haproxy/haproxy/issues/3354)

### Public snapshot service

Designed and built a public snapshot service for the node databases of a large
open-source distributed network, so new nodes can start without a full initial
sync. It serves tens of terabytes free and without bandwidth limits, as
resumable downloads of up to 1 TB, from object storage behind layered CDN
caching. Later versions produced consistent snapshots in Python pipelines on
Argo Workflows and verified each one by starting a node from it.

Downloads of the largest database files, over 40 GB each, kept failing. The
object storage sometimes answered with `206 Partial Content` that no client had
asked for, and the CDN edge cached these replies and served them to later
clients; a cache rule now excludes them. On the client side, `rclone` defaults
caused endless download loops: temporary copies needed more than 100 GB of extra
disk, and HTTP sources without modification times made it fetch finished files
again. Fixed the restore code in the Ansible role and the Helm chart.

Asked the database maintainers for a limit on table file size. Archive databases
kept table files of 160-200 GB, which made retries expensive, ruled out
incremental backups by modification time, and exceeded CDN object size limits.

* [Public snapshot service](https://snapshots.polkadot.io/)
* [paritytech/ansible-polkadot#42 - Fix HTTP snapshot restoration in the node role](https://github.com/paritytech/ansible-polkadot/pull/42)
* [paritytech/helm-charts#283 - Fix snapshot downloads in the node Helm chart](https://github.com/paritytech/helm-charts/pull/283)
* [paritytech/parity-db#219 - Request a limit on table file size](https://github.com/paritytech/parity-db/issues/219)

### libnfs

Fixed RPC payload alignment and empty-field encoding, added safe asynchronous
retries for NFSv4.2 `NFS4ERR_DELAY` replies, and mapped stale NFSv4 filehandles
to `ESTALE` so filesystem adapters can revalidate them. Validated with
regression tests, sanitizers, and real NFS workloads.

* [sahlberg/libnfs#608 - Preserve alignment of allocated ZDR payloads](https://github.com/sahlberg/libnfs/pull/608)
* [sahlberg/libnfs#609 - Retry NFS4ERR_DELAY replies when safe to replay](https://github.com/sahlberg/libnfs/pull/609) (applied upstream with minor fixes)
* [sahlberg/libnfs#610 - Avoid NULL memcpy for zero-length byte payloads](https://github.com/sahlberg/libnfs/pull/610)
* [sahlberg/libnfs#614 - Map NFSv4 stale filehandles to `ESTALE`](https://github.com/sahlberg/libnfs/pull/614)

### fuse-nfs

Kept idle NFSv4.2 sessions alive with a background libnfs service loop.
Validated concurrency, shutdown, and error handling with regression tests,
sanitizers, and real NFS workloads.

* [sahlberg/fuse-nfs#53 - Keep NFS sessions alive while the mount is idle](https://github.com/sahlberg/fuse-nfs/pull/53)

### Xpra

Contributed 14 merged changes covering native Wayland behavior, linkage, and
regression tests, build detection, hardware video decoding, notifications,
printing, input and protocol error handling, and Debian package upgrades.

* [Xpra-org/xpra#5002 - Fix Wayland frame callbacks for empty-damage commits](https://github.com/Xpra-org/xpra/pull/5002)
* [Xpra-org/xpra#5003 - Link Wayland helper extensions against `wayland-server`](https://github.com/Xpra-org/xpra/pull/5003)
* [Xpra-org/xpra#5004 - Detect libyuv installations without pkg-config metadata](https://github.com/Xpra-org/xpra/pull/5004)
* [Xpra-org/xpra#5005 - Register the native libva video decoder](https://github.com/Xpra-org/xpra/pull/5005)
* [Xpra-org/xpra#5013 - Fix Debian upgrades to the split Wayland package](https://github.com/Xpra-org/xpra/pull/5013)
* [Xpra-org/xpra#5021 - Test Wayland configure client properties](https://github.com/Xpra-org/xpra/pull/5021)
* [Xpra-org/xpra#5116 - Fix notifications with system tray forwarding disabled](https://github.com/Xpra-org/xpra/pull/5116)
* [Xpra-org/xpra#5119 - Fix `printing-info` never exiting without a system D-Bus](https://github.com/Xpra-org/xpra/pull/5119)
* [Xpra-org/xpra#5130 - Don't refresh a Wayland window destroyed while mapping it](https://github.com/Xpra-org/xpra/pull/5130)
* [Xpra-org/xpra#5138 - Publish the opaque region of Wayland popups](https://github.com/Xpra-org/xpra/pull/5138)
* [Xpra-org/xpra#5147 - Ignore notification replies from disconnected clients](https://github.com/Xpra-org/xpra/pull/5147)
* [Xpra-org/xpra#5152 - Don't send unmapped mouse wheel buttons](https://github.com/Xpra-org/xpra/pull/5152)
* [Xpra-org/xpra#5155 - Apply the packet error policy to short packets](https://github.com/Xpra-org/xpra/pull/5155)
* [Xpra-org/xpra#5161 - Close the signal watcher stdin when the exit request fails](https://github.com/Xpra-org/xpra/pull/5161)

### Prometheus Community Helm Charts

Added backward-compatible URL-path templating to the `prom-label-proxy` Helm
chart. Multiple proxy deployments can now share one hostname and use separate
paths without breaking existing values files.

* [prometheus-community/helm-charts#4482 - Add `prom-label-proxy` path templating](https://github.com/prometheus-community/helm-charts/pull/4482)

## Polkadot infrastructure automation

The following changes were merged into the public
[`paritytech/ansible-polkadot`](https://github.com/paritytech/ansible-polkadot)
collection. Published on
[Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/paritytech/chain/)
as `paritytech.chain`, it has more than 7,000 downloads and is still developed
by Parity. Most related production infrastructure remained in private Parity
repositories.

### Node deployment

* [#2 - Refactor the node role for community use and backport production features](https://github.com/paritytech/ansible-polkadot/pull/2)
  Reworked user and data-path management, task layout, tags, variables, tests,
  documentation, Prometheus annotations, and snapshot restoration from GCP.

* [#43 - Refactor separate worker-binary deployment](https://github.com/paritytech/ansible-polkadot/pull/43)
  Removed hidden coupling between binary variables, allowed independent HTTP
  roots and filenames, and prevented collisions on multi-node hosts.

### Testing and delivery

* [#32 - Publish the `ws_health_exporter` role and extend Molecule CI](https://github.com/paritytech/ansible-polkadot/pull/32)
  Published the exporter as a reusable role and extended GitHub Actions to test
  multiple roles. Added LXD-based Molecule tests where Docker did not provide
  enough operating system and systemd isolation.

* [#47 - Add collection CI/CD and release publishing](https://github.com/paritytech/ansible-polkadot/pull/47)
  Added role-based Molecule tests, collection validation, release automation,
  and publication to Ansible Galaxy.

### Snapshots, backups, and node state

* [#36 - Add HTTP snapshot restoration to the node role](https://github.com/paritytech/ansible-polkadot/pull/36)
  Added metadata-driven snapshot restoration over HTTP. The role uses `rclone`
  to read a file manifest and download files in parallel, with Molecule coverage.

* [#48 - Add `node_backup` and `state_exporter` roles](https://github.com/paritytech/ansible-polkadot/pull/48)
  Added reusable roles for node backups and host-level process monitoring. The
  exporter discovers Polkadot and Substrate processes, resolves chain and data
  paths, reports thread, CPU, memory, and command-line data, and reconstructs
  complete validator or collator session keys from the node keystore. It exposes
  the results as labelled Prometheus metrics.

* [#65 - Extend and fix `state_exporter` session-key handling](https://github.com/paritytech/ansible-polkadot/pull/65)
  Added support for more session-key formats, including BEEFY-era keys, and for
  keys created or injected at different times.

### Proxy and monitoring services

* [#62 - Add `nginx` and `nginx_exporter` roles](https://github.com/paritytech/ansible-polkadot/pull/62)
  Added reusable roles for Nginx and its Prometheus exporter. Also updated the
  supported Ansible version and rebuilt linting and Molecule CI after the old
  LXD Molecule driver became unmaintained.

## Operational tooling

These tools were developed in the public `paritytech/scripts` repository. The
repository is archived, but the original pull requests remain available.

### Substrate WebSocket health exporter

Turned a Substrate health checker into a service that monitors several
WebSocket endpoints and exposes Prometheus metrics, HTTP health checks, and
Kubernetes probes. It checks RPC access, peers, sync state, block-height drift,
and minimum block-production rate. Diagnostic logs identify the failed check.

* [paritytech/scripts#533 - Extend `ws-health-exporter`](https://github.com/paritytech/scripts/pull/533)
* [paritytech/scripts#592 - Add block-rate checks and diagnostic logging](https://github.com/paritytech/scripts/pull/592)
* [paritytech/ansible-polkadot#32 - Publish its Ansible deployment role](https://github.com/paritytech/ansible-polkadot/pull/32)

### Streaming Cloud SQL database dumper

Built a containerized Python utility that runs configurable read-only queries
against GCP Cloud SQL PostgreSQL and streams CSV rows directly to Google Cloud
Storage. It avoids a full intermediate dump in memory or on disk and supports
IAM-based Cloud SQL connections.

* [paritytech/scripts#603 - Add `db-dumper`](https://github.com/paritytech/scripts/pull/603)
* [paritytech/scripts#606 - Add logging and correct dump-file naming](https://github.com/paritytech/scripts/pull/606)

## Novasama infrastructure tooling

These tools are developed in the public
[`novasamatech/infra-tooling`](https://github.com/novasamatech/infra-tooling)
repository.

### Cloudflare analytics exporter

Exports Cloudflare HTTP analytics to Prometheus as closed 5-minute buckets. A
bucket is exported only when its groups add up to independently read totals,
and sampled responses under load are detected and read again, so data can
arrive late but never wrong. Replicas emit identical series and timestamps for
deduplication in the metrics store, and each one rebuilds its state after a
lost or corrupt database. Offline tests run against an in-memory fake of the
Cloudflare API that reproduces sampled responses.

* [novasamatech/infra-tooling#34 - Closed 5-minute buckets with self-healing storage](https://github.com/novasamatech/infra-tooling/pull/34)
* [Design notes](https://github.com/novasamatech/infra-tooling/blob/main/cf-exporter/README.md)

### PostgreSQL backup and migration tool

Built `sqdb`, a containerized tool for per-database backups and for moves
between managed PostgreSQL instances and major versions, where the indexer
databases have no superuser or WAL access. Dumps stream to S3 as multipart
uploads without local disk, and a failed upload never leaves a truncated
object. A four-phase restore creates indexes on empty tables before loading
data, avoiding the index builds that made a standard restore fail after many
hours. Each restore is checked with a DDL probe before it and with catalog and
row verification after it, also for dumps taken while the source keeps writing.
The tool was rehearsed against a 37-database configuration.

* [novasamatech/infra-tooling#30 - Add the `sqdb` migration tool and a generic CI](https://github.com/novasamatech/infra-tooling/pull/30)
* [novasamatech/infra-tooling#32 - Restore into targets with pre-installed extensions](https://github.com/novasamatech/infra-tooling/pull/32)
* [novasamatech/infra-tooling#33 - Verify dumps of live sources](https://github.com/novasamatech/infra-tooling/pull/33)
* [Restore design](https://github.com/novasamatech/infra-tooling/blob/main/subquery-db/README.md)

## Novasama SubQuery work

Novasama's indexers are streaming data pipelines in TypeScript: each one
processes a network's history block by block, writes the result to PostgreSQL,
and serves it to the mobile apps through GraphQL. The work below covers their
move to company infrastructure and changes to the Polkadot, Kusama, Asset Hub,
governance, staking, proxy, and multisig indexers.

### Self-hosted indexing platform

The indexers first ran on SubQuery's managed service. Built their deployment on
the company's own Kubernetes infrastructure and moved them there, extending the
indexers' code where the move needed it. Each indexer scales vertically and
horizontally: a full reindex after a schema change can reprocess hundreds of
gigabytes from the first block, and scaling up and out for the run shortens it.
Their dedicated managed PostgreSQL instance was tuned under indexing load
instead of being replaced with a larger one. The mobile app configuration
switched to the new endpoints in development first, then in production.

The four indexer projects hold 145 network configurations, and 51 of the 98
networks in the production app configuration read from company-run indexers.

* Indexer projects: [subquery-history](https://github.com/novasamatech/subquery-history), [subquery-accounts](https://github.com/novasamatech/subquery-accounts), [subquery-staking](https://github.com/novasamatech/subquery-staking), [subquery-governance](https://github.com/novasamatech/subquery-governance)
* [novasamatech/nova-utils#3791 - Move the history endpoint to self-hosted indexers](https://github.com/novasamatech/nova-utils/pull/3791)
* [Production network configuration](https://github.com/novasamatech/nova-utils/blob/master/chains/v23/chains.json)

### Production decoding incident

A new transaction format on one network crashed production indexing. Fixed it
with a decoder driven by each network's own metadata, ported it to four indexers
and three networks, and verified it by replaying real blocks inside the SubQuery
runtime and comparing staging with production results.

* [novasamatech/subquery-accounts#97 - Fix Asset Hub v5 decoding across networks](https://github.com/novasamatech/subquery-accounts/pull/97)
* [novasamatech/subquery-accounts#99 - Decode General-v5 extrinsics across all networks](https://github.com/novasamatech/subquery-accounts/pull/99)

### Polkadot.js v16 and runtime types

Moved the projects to `@polkadot/*` v16 and fixed compatibility issues in
runtime types, module formats, metadata, and Asset Hub type boundaries. This
included compatible dependency pins, TypeScript `node16` module resolution,
updated staking and runtime lookup types, and corrected `NovaAssetId` ranges for
Polkadot, Kusama, and Westend Asset Hubs.

* [novasamatech/subquery-staking#99 - Migrate staking indexer code to Polkadot.js v16](https://github.com/novasamatech/subquery-staking/pull/99)
* [novasamatech/subquery-accounts#87 - Update dependencies and Asset Hub chain types](https://github.com/novasamatech/subquery-accounts/pull/87)

### Cross-network decoding and historical multisig

Fixed block decoding across networks and runtime generations. The change
corrected gaps and shadowing in `NovaAssetId` definitions, restored old Kusama
multisig activity, handled legacy event layouts, and avoided address decoding
that required unavailable cryptographic runtime support inside the SubQuery
sandbox.

Added diagnostic scripts to scan historical Asset Hub metadata and signed
extensions across spec-version changes before deployment.

* [novasamatech/subquery-accounts#88 - Fix block decoding across networks after the Polkadot.js v16 upgrade](https://github.com/novasamatech/subquery-accounts/pull/88)

### Call traversal and address derivation

Updated the shared call visitor that walks nested Substrate calls and derives
their effective origins. One change avoids redundant SS58 checksum validation
for addresses already read from validated chain data. This prevents checksum
failures when WebAssembly crypto is not initialized inside the SubQuery sandbox.

A later change fixed EVM multisig address generation for nested calls with mixed
EVM and Substrate signatories. It avoids checksum and Keccak-dependent code
paths and includes regression tests for both address families.

* [novasamatech/subquery-call-visitor#34 - Use checksum-independent address decoding in the SubQuery sandbox](https://github.com/novasamatech/subquery-call-visitor/pull/34)
* [novasamatech/subquery-call-visitor#36 - Fix EVM multisig address derivation for nested calls](https://github.com/novasamatech/subquery-call-visitor/pull/36)

### Pure-proxy address reconstruction

Fixed an indexer loop affecting pure proxies created during the relay-chain to
Asset Hub migration. For `proxy.createPure` calls with a historical `when`
value, the derivation inputs come from the event payload. The indexer used the
current block envelope instead and produced the wrong address.

The handler now reads the event values and uses the block envelope only for
older event formats. The change includes validation, a regression test,
troubleshooting notes, and a rootless Podman workflow for reproducing CI.

* [novasamatech/subquery-accounts#92 - Use event payload values for pure-proxy derivation](https://github.com/novasamatech/subquery-accounts/pull/92)

### SubQuery sandbox and Node.js compatibility

Fixed a governance indexer failure where transitive cryptographic dependencies
needed `TextEncoder` and `TextDecoder`, but the SubQuery webpack sandbox did not
provide them. Added a conditional polyfill, moved CI and containers to Node.js
24, aligned Polkadot dependency versions, updated the shared call visitor, and
added structured handler logs.

* [novasamatech/subquery-governance#48 - Fix TextEncoder support and upgrade to Node.js 24](https://github.com/novasamatech/subquery-governance/pull/48)
