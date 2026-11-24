# Personal projects

These repositories grew out of practical work. Some run as operational tools;
others provide reusable building blocks for production infrastructure that
remains private.

## Debian Kernel Current

[`kogeler/dkc-linux`](https://github.com/kogeler/dkc-linux) brings the newest
authenticated Debian Sid kernel to Debian 13 without replacing its userspace.
It produces installable packages for `x86-64-v2` and `x86-64-v3` with
Debian-packaged Clang and ThinLTO, then publishes them through a signed APT
repository. A scheduled pipeline checks Debian four times a day and releases
each new source version without manual steps.

The release pipeline verifies source provenance, dependencies, recorded Kbuild
commands, and generated machine code. It also tests package installation, KVM
boot, kernel selftests, DKMS modules, removal, and fallback to the stock Debian
kernel before publication. Build, signing, storage, and public delivery use
separate trust boundaries.

* [Repository](https://github.com/kogeler/dkc-linux)
* [Architecture](https://github.com/kogeler/dkc-linux/blob/main/docs/ARCHITECTURE.md)
* [Site](https://dkc-linux.romancello.net/)

## Remote SSH MCP

[`kogeler/remote-ssh-mcp-server`](https://github.com/kogeler/remote-ssh-mcp-server)
lets Codex, Claude Code, and other MCP clients work on remote Linux systems
without a raw terminal. Each explicit `connect` authenticates once and opens a
keyed session with its own OpenSSH master for inspection, bounded commands,
passwordless sudo, and background rsync transfers.

One server process can hold sessions for several agents and remote servers, at
most one per server. Every operation must present its session's secret key, and
a lost master never reconnects silently. Large files move outside model context,
with progress, cancellation, resume, SHA-256 verification, and atomic
publication. Tests use isolated Podman targets with real OpenSSH, rsync, sudo,
master loss, parallel sessions, and both ephemeral and hardware-backed keys.

* [Repository](https://github.com/kogeler/remote-ssh-mcp-server)
* [Architecture](https://github.com/kogeler/remote-ssh-mcp-server/blob/main/doc/architecture.md)
* [Site](https://remote-ssh-mcp-server.romancello.net/)

## SSH Wrapper

[`kogeler/ssh-wrapper`](https://github.com/kogeler/ssh-wrapper) is a typed Python
library that manages one native OpenSSH ControlMaster and shares it across
commands. It preserves OpenSSH authentication and host-key policies while
requiring every channel to use the existing connection.

A heartbeat supervisor ties a remote process group to its owner and cleans it
up on owner loss. A lost master is never reconnected; the caller starts a new
one. The runtime uses only the Python standard library.

* [Repository](https://github.com/kogeler/ssh-wrapper)
* [Architecture](https://github.com/kogeler/ssh-wrapper/blob/main/docs/maintenance/architecture.md)
* [PyPI](https://pypi.org/project/ssh-wrapper/)
* [Site](https://kogeler.github.io/ssh-wrapper/)

## Elsewindow

[`kogeler/elsewindow`](https://github.com/kogeler/elsewindow) runs a remote Linux
GUI application and displays its windows locally through Xpra. Rendering stays
on the remote Wayland compositor and GPU, while picture, input, clipboard, and
notification traffic share one SSH connection managed by `ssh-wrapper`.

Sessions clean up their own processes and runtime files. Optional systemd user
services keep applications running across disconnects. Each session owns a
private notification bus and desktop portal for remote notifications and file
dialogs; missing optional support disables only that feature with a warning.
Live tests use rootless Podman to check visible windows, picture updates,
connection loss, resumption, and cleanup.

* [Repository](https://github.com/kogeler/elsewindow)
* [Architecture](https://github.com/kogeler/elsewindow/blob/main/doc/architecture.md)
* [PyPI](https://pypi.org/project/elsewindow/)
* [Site](https://kogeler.github.io/elsewindow/)

## Cluster API Kubernetes platform on LXC/LXD

[`kogeler/k8s-lab`](https://github.com/kogeler/k8s-lab) builds a self-hosted
Cluster API platform on a Debian or Ubuntu host. Kubernetes nodes run in
unprivileged LXC/LXD system containers. A temporary k3s cluster installs Cluster
API and CAPN, then hands management to a self-hosted management cluster.

Ansible roles own the host, LXD, and the management cluster, Terraform owns
each workload cluster, and Helm charts deliver Cluster API resources, Calico,
and MetalLB. Helm-test gates check convergence, the dual-stack CNI, and load
balancing, and a failed gate fails the run. Molecule runs the whole flow on a
Vagrant/libvirt VM and in GitHub Actions, including a second run that must
change nothing.

* [Repository](https://github.com/kogeler/k8s-lab)
* [Architecture](https://github.com/kogeler/k8s-lab/blob/main/doc/architecture.md)
* [Site](https://k8s-lab.romancello.net/)

## `mini-pig` Ansible collection

[`kogeler/mini-pig-ansible-collection`](https://github.com/kogeler/mini-pig-ansible-collection)
builds and operates a self-hosted bare-metal stack. Its roles cover Linux host
bootstrap, Podman and systemd workloads, WireGuard, Nginx SNI routing,
Cloudflare DDNS, Telegraf, health monitoring, and Kubernetes node integration.
Proxy roles deploy NaiveProxy and sing-box AnyTLS behind HAProxy with SNI and
ALPN routing, and a Telegram MTProxy either directly or behind HAProxy with
per-domain ACME certificates.

The firewall role manages isolated `iptables` chains or native IPv4 and IPv6
`nftables` tables without replacing rules owned by Kubernetes, Docker, libvirt,
or Podman. It supports atomic updates, drift recovery, backend migration, and
randomized SNAT. Molecule scenarios test migrations, idempotency, recovery,
foreign chains, and packet-level SNAT distribution. Proxy roles are tested in
nested Podman with real client traffic and live Telegram protocol probes.

* [Repository](https://github.com/kogeler/mini-pig-ansible-collection)

## Joplin notes and knowledge base tools

### Joplin MCP server and Markdown sync

[`kogeler/joplin-md-sync`](https://github.com/kogeler/joplin-md-sync) is a
self-hosted MCP server for a Joplin knowledge base. Hosted Claude connects
through OAuth built into the server, with sign-in limited to one configured
GitHub account, and local clients use stdio or Streamable HTTP MCP. Every
interface shares one tool registry for notes, notebooks, tags, and resources
and goes through the Joplin Data API, never the Joplin database.

One installer deploys headless Joplin Terminal, recurrent sync, and the server
as rootless systemd user services and keeps the Data API on loopback. For
coding agents that need reviewable files, a separate two-way Markdown sync uses
three-way comparison, conflict bundles, verified writes, and a run journal.
Releases go to PyPI, GitHub Releases with native executables, and the official
MCP Registry.

* [Repository](https://github.com/kogeler/joplin-md-sync)
* [Architecture](https://github.com/kogeler/joplin-md-sync/blob/main/docs/maintenance/ARCHITECTURE.md)
* [PyPI](https://pypi.org/project/joplin-md-sync/)
* [Site](https://joplin-mcp.romancello.net/)

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

## FMI integration for Home Assistant

[`kogeler/fmi-hass-custom`](https://github.com/kogeler/fmi-hass-custom) is a
maintained Home Assistant integration for Finnish Meteorological Institute
forecasts, station observations, lightning, sea-level data, and official
weather warnings. It is listed as FMI Weather in the default catalog of HACS
and the Home Assistant Marketplace, and adds map-based setup and
reconfiguration without replacing entity identity.

The maintenance work covers forecast semantics, timezone and missing-data
handling, source-specific failure isolation, registry-safe migrations,
privacy-safe diagnostics, bounded network operations, and dependency security.
CI runs offline fixtures, migration and lifecycle tests, live FMI probes,
current and prerelease Home Assistant compatibility checks, and release gates.

* [Repository](https://github.com/kogeler/fmi-hass-custom)

## Operational tooling

[`kogeler/tooling`](https://github.com/kogeler/tooling) collects small services
that I use for networking, monitoring, and hardware integration. Each utility
has its own tests, packaging, and usage documentation.

The collection includes:

* [`sms-to-telegram`](https://github.com/kogeler/tooling/tree/main/sms-to-telegram/docs)
  reliably forwards multipart GSM 7-bit and UCS2 messages from a USB modem to
  Telegram. The SIM remains the queue until every destination accepts a message.
* [`traffic-masking`](https://github.com/kogeler/tooling/tree/main/traffic-masking)
  generates measured bidirectional UDP cover traffic for encrypted tunnels,
  with PSK-authenticated sessions and per-client and aggregate rate limits.
* [`cf-ddns`](https://github.com/kogeler/tooling/tree/main/cf-ddns) and
  [`one-t-exporter`](https://github.com/kogeler/tooling/tree/main/one-t-exporter)
  provide Cloudflare DDNS automation and Prometheus metrics for validator
  performance.
