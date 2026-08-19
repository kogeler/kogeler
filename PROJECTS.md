# Personal projects

These repositories grew out of practical work. Some run as operational tools;
others provide reusable building blocks for production infrastructure that
remains private.

## Debian Kernel Current

[`kogeler/dkc-linux`](https://github.com/kogeler/dkc-linux) brings the newest
authenticated Debian Sid kernel to Debian 13 without replacing its userspace.
It produces installable packages for `x86-64-v2` and `x86-64-v3` with Clang 21
and ThinLTO, then publishes them through a signed APT repository.

The release pipeline verifies source provenance, dependencies, recorded Kbuild
commands, and generated machine code. It also tests package installation, KVM
boot, kernel selftests, DKMS modules, removal, and fallback to the stock Debian
kernel before publication. Build, signing, storage, and public delivery use
separate trust boundaries.

* [Repository](https://github.com/kogeler/dkc-linux)
* [Architecture](https://github.com/kogeler/dkc-linux/blob/main/docs/ARCHITECTURE.md)
* [Validation evidence](https://github.com/kogeler/dkc-linux/blob/main/docs/VALIDATION.md)
* [Installation guide](https://github.com/kogeler/dkc-linux/blob/main/docs/USER_INSTALL.md)

## Remote SSH MCP

[`kogeler/remote-ssh-mcp-server`](https://github.com/kogeler/remote-ssh-mcp-server)
lets Codex, Claude Code, and other MCP clients work on remote Linux systems
without a raw terminal. It reuses one explicit OpenSSH control connection for
inspection, bounded commands, controlled sudo, and background rsync transfers.

The server starts disconnected and never reconnects silently. Large files move
through rsync outside model context, with progress, cancellation, resume,
SHA-256 verification, and atomic publication. Tests use isolated Podman targets
with real OpenSSH, rsync, sudo, master loss, and both ephemeral and
hardware-backed keys.

* [Repository](https://github.com/kogeler/remote-ssh-mcp-server)
* [Architecture](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/architecture.md)
* [Security model](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/security.md)

## Cluster API Kubernetes platform on LXC/LXD

[`kogeler/k8s-lab`](https://github.com/kogeler/k8s-lab) builds a self-hosted
Cluster API platform on a Debian or Ubuntu host. Kubernetes nodes run in
unprivileged LXC/LXD system containers. A temporary k3s cluster installs Cluster
API and CAPN, then hands management to a self-hosted management cluster.

Ansible prepares the host and bootstrap environment, Terraform manages workload
cluster lifecycle, and Helm deploys Cluster API resources, Calico, and MetalLB.
The project includes dual-stack networking, deployment gates, and Molecule,
Vagrant, and libvirt test environments.

* [Repository](https://github.com/kogeler/k8s-lab)
* [Architecture](https://github.com/kogeler/k8s-lab/blob/main/doc/02-architecture.md)
* [Rendered documentation](https://k8s-lab.romancello.net/)

## `mini-pig` Ansible collection

[`kogeler/mini-pig-ansible-collection`](https://github.com/kogeler/mini-pig-ansible-collection)
builds and operates a self-hosted bare-metal stack. Its roles cover Linux host
bootstrap, Podman and systemd workloads, HAProxy and NaiveProxy, WireGuard,
Cloudflare DDNS, Telegraf, health monitoring, and Kubernetes node integration.

The firewall role manages isolated `iptables` chains or native IPv4 and IPv6
`nftables` tables without replacing rules owned by Kubernetes, Docker, libvirt,
or Podman. It supports atomic updates, drift recovery, backend migration, and
randomized SNAT. Molecule scenarios test migrations, idempotency, recovery,
foreign chains, and packet-level SNAT distribution.

* [Repository](https://github.com/kogeler/mini-pig-ansible-collection)
* [Firewall role](https://github.com/kogeler/mini-pig-ansible-collection/tree/main/roles/iptables)
* [HAProxy and NaiveProxy role](https://github.com/kogeler/mini-pig-ansible-collection/tree/main/roles/naive_proxy)

## Joplin notes and knowledge base tools

### Markdown sync

[`kogeler/joplin-md-sync`](https://github.com/kogeler/joplin-md-sync) provides
safe two-way synchronization between Joplin notes and an ordinary Markdown
workspace. This makes the same knowledge base available in Joplin, local
Markdown tools, and coding agent workspaces. Three-way comparison, explicit
conflict bundles, verified writes, and a run journal protect changes and support
recovery.

The command line interface has deterministic JSON output and stable exit codes.
MCP and authenticated REST Actions expose notes, notebooks, tags, and resources
without direct access to the Joplin database. A repository template gives
coding agents a guarded pull, edit, diff, dry-run, and push workflow. The
runtime has no third-party Python dependencies.

* [Repository](https://github.com/kogeler/joplin-md-sync)
* [Architecture](https://github.com/kogeler/joplin-md-sync/blob/main/docs/ARCHITECTURE.md)
* [Documentation](https://joplin-mcp.romancello.net/)

### OneNote and Joplin migration

[`kogeler/joplin-importer`](https://github.com/kogeler/joplin-importer) compares
read-only OneNote and Joplin snapshots, then converts a complete OneNote backup
into a deterministic Joplin export. It reports ambiguous matches in HTML, JSON,
and CSV without changing live data.

An export uses a checksum-valid source snapshot, an approved immutable plan,
and a no-mutation dry run. It stages the complete managed tree, verifies the
result, and only then promotes it. Partial or fuzzy merges are outside the
public command line interface.

* [Repository](https://github.com/kogeler/joplin-importer)
* [Architecture](https://github.com/kogeler/joplin-importer/blob/main/docs/ARCHITECTURE.md)
* [Workspace format](https://github.com/kogeler/joplin-importer/blob/main/docs/WORKSPACE_FORMAT.md)

## FMI integration for Home Assistant

[`kogeler/fmi-hass-custom`](https://github.com/kogeler/fmi-hass-custom) is a
maintained Home Assistant integration for Finnish Meteorological Institute
forecasts, station observations, lightning, and sea-level data. It supports
current Home Assistant releases and adds map-based setup and reconfiguration
without replacing entity identity.

The maintenance work covers forecast semantics, timezone and missing-data
handling, source-specific failure isolation, registry-safe migrations,
privacy-safe diagnostics, bounded network operations, and dependency security.
CI runs offline fixtures, migration and lifecycle tests, live FMI probes,
current and prerelease Home Assistant compatibility checks, and release gates.

* [Repository](https://github.com/kogeler/fmi-hass-custom)
* [User guide](https://github.com/kogeler/fmi-hass-custom/blob/master/docs/USER_GUIDE.md)
* [Maintenance changelog](https://github.com/kogeler/fmi-hass-custom/blob/master/CHANGELOG.md)

## Operational tooling

[`kogeler/tooling`](https://github.com/kogeler/tooling) collects small services
that I use for networking, monitoring, hardware integration, and Polkadot
operations. Each utility has its own tests, packaging, and usage documentation.

The collection includes:

* [`sms-to-telegram`](https://github.com/kogeler/tooling/tree/main/sms-to-telegram)
  reliably forwards multipart GSM 7-bit and UCS2 messages from a USB modem to
  Telegram. The SIM remains the queue until every destination accepts a message.
* [`traffic-masking`](https://github.com/kogeler/tooling/tree/main/traffic-masking)
  generates measured bidirectional UDP cover traffic for encrypted tunnels,
  with runtime control and Prometheus metrics.
* [`polkadot-nominations`](https://github.com/kogeler/tooling/tree/main/polkadot-nominations)
  is a strictly typed TypeScript CLI for validator nominations, self-stake
  statistics, and historical reward analysis on Polkadot Asset Hub.
* [`cf-ddns`](https://github.com/kogeler/tooling/tree/main/cf-ddns) and
  [`one-t-exporter`](https://github.com/kogeler/tooling/tree/main/one-t-exporter)
  provide Cloudflare DDNS automation and Prometheus metrics for validator
  performance.
