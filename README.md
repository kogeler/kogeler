# About

I'm a geek at heart. I started programming and hacking at 13, mostly with
open-source software, taking things apart just to see how they work. Around the
same time, I started caring about security and privacy.

That curiosity became my career. In telecommunications, I learned networking
hands-on, designing networks and keeping them running myself, from fibre optics
to the mobile core. In fintech, I learned how large companies approach security
and process. Today I work as an SRE across data-centre, cloud-native, and
blockchain environments. I design, build, and operate infrastructure for
stateful and distributed systems, from bare metal to virtualization and
Kubernetes clusters. My work also covers networking, storage, observability,
and release automation.

I build tools that let coding agents work with remote systems and knowledge
bases. I focus on controlled tool access, verifiable results, and recovery
from failed operations, treating model output as untrusted input. Security is
not only part of my job: I protect my own data the way I would protect a
production system, with hardware keys, encryption, signing, and backups.

## How I work

I take ownership from architecture through production. I turn operational
needs into reproducible systems, automate delivery, and test upgrades and
recovery paths. I stay with incidents until the root cause is understood,
tracing failures across application, protocol, operating system, network,
storage, and hardware layers. Reproducible failures become regression tests.

A few principles I rely on:

* The best way to solve a problem is to discuss it together.
* Check existing solutions first.
* The simpler the implementation, the better.
* A solution is only effective if it takes related tasks into account.
* If research isn't a challenge, it isn't research.
* Any task can be done in the best possible way, even if it wasn't planned
  like that from the start.

## Upstream contributions

I believe the open-source philosophy built the modern world. It's not perfect,
but it really works, and I contribute back where I can.

My upstream work includes diagnosing a production HTTP/2 failure in HAProxy,
fixing serialization and session handling in libnfs and fuse-nfs, and improving
Wayland support, video decoding, and packaging in Xpra. Other contributions
cover Polkadot deployment automation and SubQuery indexer fixes.

[Upstream work and public contributions](UPSTREAM.md).

## Engineering stories

Some problems take more than a quick fix: UDP lost in one direction inside a
cloud provider's backbone, padding bytes that made HAProxy drop every stream on
an HTTP/2 connection, a filesystem bottleneck reproduced before production ran
into it, multi-host tests three isolation layers deep yet light enough for a
developer's laptop. These stories follow each one down to its cause, and end
with how AI agents are changing my everyday work.

[Stories from my work](STORIES.md).

## Personal projects

My day-to-day work spans many employer repositories. Some of them are public,
but most of the code lives in private ones, so I don't list them here. Below are
smaller projects I maintain myself. I sometimes use them as a foundation for
solutions I build at work.

* [Debian Kernel Current](https://github.com/kogeler/dkc-linux) builds Debian Sid
  kernel packages for Debian 13 with Clang and ThinLTO, tests them in KVM, and
  publishes them through a signed APT repository.
* [k8s-lab](https://github.com/kogeler/k8s-lab) provides a self-hosted Cluster
  API platform with declarative workload clusters in unprivileged LXC/LXD
  containers on a Debian or Ubuntu host.
* [Remote SSH MCP](https://github.com/kogeler/remote-ssh-mcp-server) gives coding
  agents bounded tools for remote Linux administration and verified, resumable
  file transfers outside model context.
* [SSH Wrapper](https://github.com/kogeler/ssh-wrapper) is a typed Python library
  for sharing one OpenSSH connection across commands. Its heartbeat supervisor
  cleans up owned remote processes when their caller disappears.
* [Elsewindow](https://github.com/kogeler/elsewindow) shows remote Linux application
  windows locally through Xpra over SSH. Rendering stays on the remote GPU;
  optional persistent sessions survive client disconnects.
* [mini-pig](https://github.com/kogeler/mini-pig-ansible-collection) provides Ansible
  roles for bare-metal hosts, networking, and services. Its firewall rules
  coexist with Kubernetes, Docker, libvirt, and Podman.
* [Joplin tools](PROJECTS.md#joplin-notes-and-knowledge-base-tools) provide
  two-way Markdown sync, MCP access to Joplin notes, and deterministic
  migration from OneNote.
* [FMI for Home Assistant](https://github.com/kogeler/fmi-hass-custom) brings
  Finnish weather forecasts, observations, lightning, and sea-level data into
  Home Assistant. This maintained fork adds tested migrations and release
  compatibility checks.

[More personal projects and technical details](PROJECTS.md).
