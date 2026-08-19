# Upstream work and public contributions

Most of my production work is private or employer-owned. These public issues
and merged changes show how I debug failures, build operational tooling, and
maintain distributed systems.

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

### Prometheus Community Helm Charts

Added backward-compatible URL-path templating to the `prom-label-proxy` Helm
chart. Multiple proxy deployments can now share one hostname and use separate
paths without breaking existing values files.

* [prometheus-community/helm-charts#4482 - Add `prom-label-proxy` path templating](https://github.com/prometheus-community/helm-charts/pull/4482)

## Polkadot infrastructure automation

The following changes were merged into the public
[`paritytech/ansible-polkadot`](https://github.com/paritytech/ansible-polkadot)
collection. Most related production infrastructure remained in private Parity
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

## Novasama SubQuery work

These TypeScript and JavaScript changes cover Polkadot, Kusama, Asset Hub,
governance, staking, proxy, and multisig indexers.

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
