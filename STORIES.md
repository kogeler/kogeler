# Engineering stories

<!-- CVs and cover letters link to the story headings, and a sent CV keeps the address: do not rename or remove a heading. One paragraph per line, do not rewrap lines. -->

Stories from my work, in more detail than a CV has room for.
Each one starts with a problem, follows it to the layer where the cause or the hard part was, and shows what I had to learn on the way.
Employer code and configurations are private, and I leave out the names of third-party providers. Each story ends with its public trace or says what is not public.

| Story | Theme | Where | Goes into |
| --- | --- | --- | --- |
| [One-way UDP loss in a provider backbone](#one-way-udp-loss-in-a-provider-backbone) | Troubleshooting | Novasama | Inter-region UDP, WireGuard overlay, packet captures |
| [Leftover padding in the HAProxy HTTP/2 demuxer](#leftover-padding-in-the-haproxy-http2-demuxer) | Troubleshooting | Own project | HTTP/2 framing, HAProxy traces and C code |
| [Predicting a Btrfs bottleneck under RocksDB](#predicting-a-btrfs-bottleneck-under-rocksdb) | Troubleshooting | Parity | LSM-tree compaction, Btrfs copy-on-write |
| [Multi-host Molecule tests in nested Podman](#multi-host-molecule-tests-in-nested-podman) | Infrastructure as code | Novasama | Nested namespaces, UID/GID mappings, cgroup delegation |
| [Self-service deployments with coding agents](#self-service-deployments-with-coding-agents) | AI | Novasama | GitOps, Helm, specifications for agents |
| [SDD framework for backend coding agents](#sdd-framework-for-backend-coding-agents) | AI | Novasama | Architecture, testing and observability rules for agents |
| [Agents in daily operations](#agents-in-daily-operations) | AI | Novasama | Logs, telemetry, dashboards and alerts through MCP |

## Troubleshooting

Most infrastructure troubleshooting is unglamorous, especially on bare metal: flaky SSD controllers, faulty RAM, and the occasional long battle to convince a cloud provider that the problem is actually on their side.
The stories below are the exceptions, each interesting for a different reason.

### One-way UDP loss in a provider backbone

**Intermittent HTTP 500 alerts in a multi-provider Kubernetes cluster led to UDP lost in one direction between some of a cloud provider's regions: a backbone fault that the provider confirmed and fixed.**

Novasama Technologies · Kubernetes, Kilo, WireGuard, tcpdump, libp2p

**Situation.** For years, our libp2p-based services kept performing worse in one cloud provider's Polish region. I never investigated it properly; I just stopped provisioning cloud bare-metal servers there. Then I added a pool of VMs from that region to an existing Kubernetes cluster, and soon we were getting intermittent HTTP 500 alerts, although individual pods often looked healthy.

**Digging in.** The cluster could take worker nodes from any provider: Kilo, its CNI, joined them into a full-mesh WireGuard network. The provider gave no direct access to its managed worker VMs, so I deployed privileged debugging pods in several regions and inspected traffic with tcpdump on different network interfaces. The pattern was very specific: between certain regions, UDP packets were lost completely in one direction, while TCP worked normally. WireGuard runs over UDP, and that explained the failures in the overlay.

**Result.** I collected evidence that the problem was not specific to our setup. After a lengthy investigation, the provider confirmed a systemic issue in its inter-region backbone network that affected several of its infrastructure products, and eventually fixed it. That also explained the libp2p degradation I had been working around for years.

**Takeaway.** A workaround had hidden the same fault for years; a strange symptom is worth a proper investigation the first time it shows up.

Not public: the cluster configuration and the support case with the provider.

### Leftover padding in the HAProxy HTTP/2 demuxer

**Every so often, all streams of a multiplexed HTTP/2 connection through HAProxy failed at once; I traced it to DATA-frame padding that HAProxy accounted for but never removed, and the maintainers fixed it in all affected branches.**

Own project · HTTP/2, HAProxy traces, Wireshark, C

**Situation.** This case comes from my own infrastructure, not my day job. I was multiplexing long-lived HTTP/2 CONNECT tunnels through HAProxy when, every so often, all streams in a connection failed at once with `GOAWAY PROTOCOL_ERROR`. It happened under real network latency and congestion and was hard to reproduce locally.

**Digging in.** Even decrypted traffic in Wireshark didn't show why HAProxy was rejecting apparently valid HTTP/2 frames. So I enabled HAProxy's internal HTTP/2 tracing and collected a large amount of diagnostic data: buffer sizes, offsets and state transitions. Matching application failures with those traces, RTT, congestion and flow-control state across several release branches ruled out the backend and showed that configuration tuning only changed the failure rate. With an LLM helping me navigate and analyze HAProxy's complex C code, I traced the failure to a subtle bug: padding bytes in DATA frames were accounted for but never removed from the input buffer. The demuxer then read the leftover padding as the next frame header and terminated the whole connection.

**Result.** I reported the findings upstream with detailed evidence. The maintainers reproduced the issue, confirmed the root cause and released a fix across all affected branches.

Public trace: [the report, haproxy#3354](https://github.com/haproxy/haproxy/issues/3354), [the upstream fix](https://github.com/haproxy/haproxy/commit/faf3e9ac3a5df7258b0abbc06b0e0378617a18e5), [my capture and analysis scripts](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/doc/scripts/README.md).

### Predicting a Btrfs bottleneck under RocksDB

**I expected Btrfs copy-on-write to become a bottleneck for large RocksDB databases and reproduced it in workload tests before production hit it; the team moved to a replication strategy that does not depend on filesystem snapshots.**

Parity Technologies · RocksDB, Btrfs, ext4, ZSTD, SSD

**Situation.** When I joined Parity, its large RocksDB-backed services, some approaching a terabyte of state, already ran on Btrfs. It was a common choice among companies with similar workloads: fast snapshots, writable clones for provisioning replicas, and transparent ZSTD compression (level 4) to save SSD space.

**Digging in.** I suspected the design would become a performance bottleneck as the databases grew, so I studied RocksDB's LSM-tree architecture, SST file lifecycle and compaction behavior, along with Btrfs copy-on-write mechanics. Then I built reproducible workload tests. Beyond a certain database size, storage latency rose sharply, although the SSDs were only about 25% full. The same workload on ext4, on the same hardware and SSDs, showed no such degradation. That isolated the problem to the Btrfs layer or its configuration. Disabling copy-on-write was no way out: it would also turn off transparent compression, and with it a key benefit of the original design.

**Result.** I presented the evidence to the team, and we migrated to a replication strategy independent of filesystem snapshots before the issue became a major production bottleneck.

**Takeaway.** A common choice still has to be tested against your own workload and the way it grows.

Not public: the workload tests and the replication setup.

## Infrastructure as code

I believe thorough testing is the most important part of reliable infrastructure as code.
Even a careful review cannot anticipate every detail of the target environment, and deployment should be the final step, not the first real test.

### Multi-host Molecule tests in nested Podman

**Multi-host Ansible tests were too heavy to run locally, so I built a Podman-in-Podman environment for Molecule that runs the production systemd units with their full hardening and without privileged containers; engineers began running the full end-to-end suite locally.**

Novasama Technologies · Ansible, Molecule, Podman, systemd, Linux namespaces, cgroups, nftables, GitHub Actions

**Situation.** We maintained Ansible roles for low-level Linux configuration and for services that ran outside Kubernetes on dedicated bare-metal servers, some of them as Podman pods under systemd. Testing them properly took several isolated Linux hosts talking over a network. Molecule with virtual machines was too heavy for developers' machines and for free GitHub Actions runners, so engineers often skipped local testing, and the development cycle slowed down.

**Digging in.** I built a lightweight Podman-in-Podman test environment for Molecule. Molecule provisions several unprivileged Podman containers, each simulating an independent Linux host with its own systemd. Ansible deploys the actual production systemd units, which apply their security restrictions and then launch the application pods through nested Podman. The main challenge was making three layers of isolation work together: the outer Podman container, systemd sandboxing and the isolation enforced by the inner Podman containers. The inner workloads had to run with the same security hardening and network configuration as on real bare-metal hosts. That took extensive troubleshooting of nested Linux namespaces, UID/GID mappings, cgroup delegation and networking, all without privileged containers, additional host permissions or bind mounts of host resources.

**Result.** The team got a reproducible environment for real multi-host end-to-end tests, light enough to run locally and within the limits of free GitHub Actions runners, with no dedicated infrastructure and no weakened production security settings. Even low-level configuration such as nftables could be tested. That is what brought the team on board: role developers readily ran the full end-to-end suite locally instead of waiting for long CI runs, because it was simpler, safer and faster to start. Development got faster, and since the tests took fewer resources both locally and in CI, the test scenarios grew quickly.

**Takeaway.** Tests get run when running them is the easiest path.

Not public: the employer roles and their tests. The same nested-Podman approach in my own Ansible collection: [Molecule scenarios of the naive_proxy role](https://github.com/kogeler/mini-pig-ansible-collection/blob/main/roles/naive_proxy/doc/contracts/verification.md#molecule-scenarios).

## AI in infrastructure work

### Self-service deployments with coding agents

**Preparing a Kubernetes deployment for a new service took considerable DevOps time, so I wrote a specification that developers' coding agents follow to prepare it themselves, while the merge stayed with the infrastructure team.**

Novasama Technologies · Argo CD, Helm, Kubernetes, GitOps, coding agents

**Situation.** When AI coding agents were only starting to emerge, I began exploring what they could do for infrastructure engineering. One opportunity was to make Kubernetes deployments self-service for developers and QA engineers. We already had mature GitOps on Argo CD, with applications discovered automatically through templates, but preparing a deployment for a new service still took considerable DevOps involvement.

**Digging in.** I adapted a general-purpose Helm chart and introduced a spec-driven development (SDD) workflow for AI agents. The specification lived in the dev deployments directory, next to existing configurations that served as examples, and defined the infrastructure conventions, constraints and mandatory validation checks. A developer gave an agent the service's source code or documentation, the agent prepared a deployment configuration by the specification, and the developer opened a pull request that only the infrastructure team could merge. Once it was approved, Argo CD discovered and deployed the application on its own. The workflow targeted a dedicated development cluster, where a configuration mistake costs far less than in production.

**Result.** Some tasks still needed manual work, mainly external secrets and infrastructure dependencies. But much of the repetitive work and of the back-and-forth was gone, while human review and infrastructure standards stayed. I also presented the approach in a short talk at a local DevOps meetup in Helsinki.

Public trace: [the meetup post about the talk](https://lnkd.in/p/eKw7fn-w). Not public: the specification and the deployment repository.

### SDD framework for backend coding agents

**Coding agents wrote infrastructure backend services fast but with too many tests, too much boilerplate and overengineered code, so I turned years of SRE experience into a shared specification framework that the agents follow.**

Novasama Technologies · Python, spec-driven development, coding agents, Podman, Docker

**Situation.** Besides infrastructure code, we regularly build infrastructure-related backend services, such as caching layers, API protection and authentication. AI agents made this work much faster, but they also tend to produce too many tests, too much documentation and boilerplate, and overengineered code. Reviews get harder, and subtle defects sometimes hide in the volume.

**Digging in.** I designed and introduced a shared SDD framework for backend development. It defines our technology stack, architecture patterns, testing strategies and coverage requirements, telemetry and observability, scalability, database usage, and synchronization patterns with message queues. In effect, I translated years of SRE experience into structured specifications that developers' agents follow consistently.

**Result.** Infrastructure requirements that used to be hard to communicate and enforce now enter the development process automatically, with less overhead and more consistent, reviewable code. Product teams adopted the framework and develop faster.

**Next.** I am building a local execution harness that lets agents start an almost complete application stack with Podman or Docker. Instead of relying only on generated tests, agents run real end-to-end tests, including integrations with external APIs through restricted development credentials. The goal is strict application-level quality gates: an agent has to verify that the actual service works before it submits code for review.

Not public: the framework, its project templates and the harness.

### Agents in daily operations

**Agents with access to logs, metrics and dashboards made routine operations several times faster, and changed how much I still do by hand.**

Novasama Technologies · MCP, Grafana, Kubernetes, logs and telemetry, coding agents

**Situation.** AI has changed my daily infrastructure work, above all troubleshooting, logging, monitoring and alerting, where much of the effort used to go into reading and tedious manual edits: when backend metrics changed, dashboards had to be edited by hand.

**Digging in.** I gave agents read-only Kubernetes access and MCP servers for logs, dashboards and metrics. With them, scripting and SDD-based specifications, I analyze large volumes of logs and telemetry, investigate anomalies, generate and update Grafana dashboards in bulk and keep alerting configurations consistent. When backend metrics change, the dashboards are updated automatically.

**Result.** Routine infrastructure operations have become much faster and easier to maintain, and the same team investigates incidents several times faster. With agents, I build a fully working infrastructure service in three days and analyze several days of logs and telemetry in two minutes; both would take me far longer without AI.

**Takeaway.** Since 2025, my team and I have almost stopped writing code and configuration by hand, and even manual troubleshooting has become rare. I'm more productive than ever and probably getting worse at doing these things myself. Skills fade without practice, and I still don't know whether to celebrate this progress or worry a little.

Not public: the agent setup, dashboards and alert rules. Public: [Remote SSH MCP](https://github.com/kogeler/remote-ssh-mcp-server), my own server that agents use at work on non-production hosts.
