# About

I design, build, and operate infrastructure for stateful and distributed
systems. My background includes data-centre, telecommunications, and blockchain
environments. My work spans bare metal and Linux hosts, cloud-native platforms,
networking, storage, observability, and release automation.

I take ownership of work from architecture through production. I turn
operational needs into reproducible systems, define trust and failure
boundaries, automate delivery, and make upgrades and recovery testable. I stay
with incidents until the root cause is understood, including when the
investigation crosses application, protocol, operating system, network,
storage, or hardware layers.

I expect infrastructure changes to be reviewable and repeatable, with a tested
recovery path. I use isolated tests and deployment gates for upgrades,
migrations, and failure paths. When a production issue can be reproduced, I add
a regression check.

I follow how cloud-native platforms and AI-assisted engineering are evolving.
In AI systems, I focus on agent orchestration, controlled tool access,
structured outputs, evaluation, and recovery. I treat model output as untrusted
input and apply the same review, testing, and operational standards to AI
components as to any other production dependency.

Most of my production work is private or employer-owned. The public work below
provides concrete examples of how I design, build, and operate systems.

## Personal projects

These repositories started from practical problems. Many are used directly as
operational tools or as reusable foundations for production infrastructure
kept in private repositories.

* [Debian Kernel Current](https://github.com/kogeler/dkc-linux) brings the newest
  authenticated Debian Sid kernel to Debian 13 without replacing its userspace.
  Its unattended release pipeline builds LLVM and ThinLTO packages, qualifies
  them in KVM, and publishes a signed APT repository.
* [k8s-lab](https://github.com/kogeler/k8s-lab) provides a self-hosted Cluster
  API platform on one Debian or Ubuntu host, using unprivileged LXC/LXD
  containers instead of virtual machines. It provisions declarative workload
  clusters and tests the complete deployment path.
* [Remote SSH MCP](https://github.com/kogeler/remote-ssh-mcp-server) lets coding
  agents work on remote Linux systems without a raw terminal. It exposes
  bounded operations over one fail-closed OpenSSH connection and moves large
  files outside model context with verified, resumable transfers.
* [mini-pig](https://github.com/kogeler/mini-pig-ansible-collection) is a
  reusable Ansible base for self-hosted bare-metal systems. It covers host
  setup, networking, VPN and proxy services, monitoring, and workloads, with a
  firewall policy that coexists with Kubernetes, Docker, libvirt, and Podman.
* [Joplin tools](PROJECTS.md#joplin-notes-and-knowledge-base-tools) connect a
  Joplin knowledge base to Markdown and coding agents. They provide safe
  two-way note sync, direct access through MCP and authenticated ChatGPT
  Actions, and deterministic migration from OneNote.
* [FMI for Home Assistant](https://github.com/kogeler/fmi-hass-custom) brings
  Finnish weather forecasts, observations, lightning, and sea-level data into
  Home Assistant. This maintained fork adds tested migrations, failure
  isolation, compatibility checks, dependency review, and automated releases.

[More personal projects and technical details](PROJECTS.md).

## Upstream contributions

My public upstream work includes production debugging, infrastructure
automation, operational services, and SubQuery indexer fixes. Each entry is
linked to the original issue or merged pull request.

[Upstream work and public contributions](UPSTREAM.md).
