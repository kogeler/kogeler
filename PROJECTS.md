# Selected personal infrastructure projects

Most of my production infrastructure is environment-specific and lives in private repositories. The projects below are public, reusable components and reference implementations derived from the same engineering approach.

## `mini-pig` Ansible collection

[`kogeler/mini-pig-ansible-collection`](https://github.com/kogeler/mini-pig-ansible-collection) is a personal Ansible collection for building and operating a self-hosted bare-metal stack across Linux networking, proxy and VPN services, observability, containers, and base operating-system configuration.

The collection deploys container workloads through Podman and systemd and contains roles for HAProxy and NaiveProxy, WireGuard, TLS routing, Cloudflare DDNS, Telegraf, systemd health monitoring, host bootstrap, and Kubernetes node integration. Non-trivial roles are covered by Ansible Molecule scenarios and CI.

### Linux firewall and network-policy management

The firewall role demonstrates lower-level Linux networking work rather than only high-level service configuration.

It manages isolated role-owned `iptables` chains or native IPv4 and IPv6 `nftables` tables without flushing or replacing rules maintained by Kubernetes, Docker, libvirt, Podman, or other host components.

Implemented functionality includes:

* static and randomized SNAT;
* atomic ruleset updates;
* configuration-drift recovery;
* migration between `iptables` and `nftables` backends.

Randomized SNAT is implemented in a dedicated native nftables chain evaluated before kube-proxy, CNI, or other `srcnat` managers. Molecule scenarios validate migrations, idempotency, interaction with foreign chains, firewall recovery, and packet-level SNAT distribution.

* [Repository](https://github.com/kogeler/mini-pig-ansible-collection)
* [`iptables` and `nftables` firewall role](https://github.com/kogeler/mini-pig-ansible-collection/tree/main/roles/iptables)
* [HAProxy and NaiveProxy deployment role](https://github.com/kogeler/mini-pig-ansible-collection/tree/main/roles/naive_proxy)

## Cluster API Kubernetes platform on LXC/LXD

[`kogeler/k8s-lab`](https://github.com/kogeler/k8s-lab) implements a self-hosted Cluster API platform on a Debian or Ubuntu bare-metal or virtual-machine host, using unprivileged LXC/LXD system containers as Kubernetes nodes.

A transient single-node k3s cluster bootstraps Cluster API and the CAPN infrastructure provider. Management responsibility is then pivoted to a self-hosted management cluster, which declaratively provisions workload clusters using Cluster API resources, kubeadm control-plane and bootstrap providers, Terraform, and Helm.

The repository contains reusable platform building blocks rather than environment-specific production configuration:

* Ansible roles for host preparation, the LXD substrate, bootstrap, Cluster API installation, pivoting, validation, and teardown;
* a Terraform module for workload-cluster lifecycle and topology;
* Helm charts for Cluster API resources, Calico, and MetalLB;
* dual-stack and IPv6 ingress networking;
* Molecule, Vagrant, and libvirt test environments;
* deployment gates validating CNI and external-network reachability.

The project was developed as a reusable architectural and automation base for production deployments. Concrete inventories, secrets, environment-specific variables, and deployment composition are maintained in separate private consumer repositories that import the public project.

* [Repository](https://github.com/kogeler/k8s-lab)
* [Architecture documentation](https://github.com/kogeler/k8s-lab/blob/main/doc/02-architecture.md)
* [Rendered documentation](https://k8s-lab.romancello.net/)
