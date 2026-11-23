<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="logo/ate-logo-with-border.svg">
  <img src="logo/ate-logo.svg" alt="Agent Substrate logo: a cybernetic green octopus orchestrating sandboxed agent puppets" width="260">
</picture>

# Agent Substrate

**Zero-trust sandboxing for millions of agents on Kubernetes.**

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Release](https://img.shields.io/badge/release-v1.0.0-brightgreen.svg)](https://github.com/Amz34/substrate/releases)
[![CI](https://img.shields.io/badge/CI-passing-brightgreen.svg)](https://github.com/Amz34/substrate/actions)
[![govulncheck](https://img.shields.io/badge/govulncheck-passing-brightgreen.svg)](https://github.com/Amz34/substrate/actions)
[![GitHub stars](https://img.shields.io/github/stars/Amz34/substrate?style=flat)](https://github.com/Amz34/substrate/stargazers)

[Quickstart](#quickstart-development) · [Security](docs/security/security.md) · [Architecture](docs/concepts/architecture.md) · [Contributing](CONTRIBUTING.md)

<a href="#what-is-agent-substrate">
  <img src="docs/assets/readme-hero.svg" alt="Agents are idle most of the time. Agent Substrate suspends idle agents and resumes them on request onto a small pool of warm worker pods." width="900">
</a>

</div>

## What is Agent Substrate?

Agent Substrate is a security-hardened, zero-trust runtime for sandboxed agents, engineered to safely run millions of agents and tools with 10x–30x higher density than standard containers. Purpose-built for the era of autonomous agents, Substrate delivers fast resume operations in less than 500ms, with more than 500 activations per second. Agents get isolated compute, identity, networking, and storage. Sensitive credentials are kept out of reach from untrusted code. It supports multiple sandbox technologies, including microVMs and gVisor, while enabling a consistent lifecycle and management of all sandboxes.

At its core, Agent Substrate maps a larger set of “actors” (stateful workloads such as agents) onto a smaller pool of ready “workers”. Because agents spend most of their lives waiting on models, tools, or humans, Substrate snapshots idle actors out of memory and restores them when a request arrives.

Substrate is a low-opinion runtime, not an agent SDK. The workloads it runs do not have to be AI agents, though autonomous, untrusted, mostly-idle agents are the target for its isolation and scheduling model. It runs on standard Kubernetes: Pods, autoscaling, NetworkPolicies. It adds agent-specific scheduling, snapshotting, identity, and network mediation. This allows agentic, inference, training, and reinforcement learning workloads to share one infrastructure plane.

## Security Model

Substrate treats every actor as untrusted and mutually hostile, so tenants can share physical nodes without sharing a security boundary. Every control operates within the cold-start and resume budget. See the [Security Guide](docs/security/security.md) for the full list of security principles and threat model.

* **Always sandboxed, VM-grade boundaries:** Every actor runs inside a dedicated [gVisor](https://gvisor.dev/) (`runsc`) or [Cloud Hypervisor](https://www.cloudhypervisor.org/) microVM sandbox with segregated compute, network, and storage. Plain Linux containers are not supported.
* **Per-agent cryptographic identity:** Every active actor receives a short-lived [SPIFFE](https://spiffe.io/) X.509 certificate by default, and the egress gateway can mint per-actor JWTs on demand when an egress policy calls for one. Both carry the actor identity and atespace (the Substrate namespace grouping), and are designed to federate with external identity systems.
* **Zero credentials in the sandbox:** Actors never hold actor identity JWTs/certs, API keys, or cloud tokens. The built-in egress gateway ([`atenet-egress`](docs/guides/egress-credential-injection.md)) injects credentials into approved outbound requests outside the sandbox, so a compromised agent has nothing to exfiltrate.
* **Default-deny networking:** Actors start with a private network stack, no inbound path except through the ingress router (`atenet-router`), no lateral actor-to-actor or control-plane access, and outbound traffic [restricted by the egress gateway](docs/guides/egress-traffic.md).
* **Pluggable policy at the edge:** The egress gateway acts as an external Policy Enforcement Point (PEP) that is fully replaceable, allowing you to attach your own authorization, DLP, content scanning, or cloud IAM policy engine without placing policy code inside the untrusted sandbox.

## Performance & Multiplexing

[![Agent Substrate Demo](https://img.youtube.com/vi/ZEzkCFJkzjY/hq1.jpg)](https://www.youtube.com/watch?v=ZEzkCFJkzjY)

*Watch the Agent Substrate cluster multiplex ~250 stateful actors across just 8 physical pods.*

This demo highlights the core developer experience and "Agentic Infrastructure" capabilities of Substrate:

1. **Actor Teleport:** Suspend and resume actors onto any available worker in the pool, when needed.
2. **Full-State Persistence:** Working memory (RAM) and filesystem state are captured in snapshots and restored intact across hibernation cycles, without losing in-flight sessions or tool state.
3. **10x–30x Oversubscription:** Multiplex a large registry of stateful actors onto a small pool of warm worker pods, paying for compute only while an agent is actively thinking or executing a tool.

To reproduce this demo in your own cluster, see the **[Counter Demo](demos/counter/README.md)**.

## Star History

<a href="https://www.star-history.com/?repos=Amz34%2Fsubstrate&type=date&legend=top-left">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=Amz34/substrate&type=date&theme=dark&legend=top-left">
    <img alt="Star history chart for Amz34/substrate" src="https://api.star-history.com/svg?repos=Amz34/substrate&type=date&legend=top-left">
  </picture>
</a>

## Framework Agnostic & Compatibility

Agent Substrate is **framework and agent harness agnostic**. Because it runs standard OCI container images inside gVisor or microVM sandboxes and enforces security outside the workload, it hosts agents built on any stack without SDK changes:

* **Agent Development Kit (ADK):** Preserves session state across invocations as actor state.
* **LangChain:** Drop-in execution environment for LangChain agents and tool calls.
* **Claude Code, Codex, and Antigravity:** High-density, stateful coding environments that keep system and filesystem state across sessions while isolating untrusted code execution.
* **Model Context Protocol (MCP):** Deploy sandboxed MCP servers as Substrate Actors with per-actor identity and egress controls.

## Ecosystem & Examples

*   **[Agent Executor](https://github.com/google/ax):** A distributed agent runtime that demonstrates building a secure, hyper-scalable agent harness on Agent Substrate.
*   **[kagent](https://github.com/kagent-dev/kagent):** A CNCF Sandbox project and Kubernetes-native framework for building, deploying, and managing AI agents that uses Agent Substrate to run sandboxed, stateful agent workloads.

## Status and compatibility

Agent Substrate has reached 1.0. We are committed to compatibility and stability of the public API, CLI, and configuration formats. We will continue to add new features, but we will not break existing APIs or configuration formats without a major version bump. See "Compatibility and schema evolution" in [CONTRIBUTING.md](CONTRIBUTING.md) for more details.

### Supported Kubernetes Releases

Currently we aim to support the [latest stable release](https://kubernetes.io/releases/) of Kubernetes, and the previous minor release.

## Developing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on contributing to the project. We welcome contributions of all kinds. Our immediate focus is on building out the core system and demos.

## Quickstart (Development)

To quickly set up the complete environment:

1. Make sure you have [Go](https://go.dev/doc/install), [`kubectl`](https://kubernetes.io/docs/tasks/tools/), and [`docker`](https://www.docker.com/) installed and configured on your dev machine. We will automatically manage other dependencies via Go, including [`kind`](https://kind.sigs.k8s.io/).
   A Linux host that will run more than about a thousand sandboxes also needs a larger kernel neighbor table (`net.ipv4.neigh.default.gc_thresh1/2/3` = 4096/16384/32768); `hack/create-kind-cluster.sh` warns when the host is at the default of 1024.

2. Run the following steps:
```shell
# create cluster and local registry
hack/create-kind-cluster.sh

# make ate-setup target the kind cluster; set this in every shell that runs
# ate-setup, including for the demo walkthroughs
export ATE_INSTALL_KIND=true

# install Substrate
go run ./cmd/ate-setup deploy ate-system --credential-provider='{"name":"k8s.io"}' \
  --ingress-allowed-spiffe-ids=spiffe://cluster.local/ns/ate-system/sa/atenet-client

# install counter demo
go run ./cmd/ate-setup deploy demo counter

# install kubectl-ate
go install ./cmd/kubectl-ate

# create a counter actor in the demo's atespace (--template names the
# actor template, resolved in the actor's atespace)
kubectl ate create actor my-counter-1 -a ate-demo-counter --template counter

# serve a local proxy on port 8000 that sends requests to the actor through the
# network router
kubectl ate proxy actor my-counter-1 -a ate-demo-counter
```

> [!IMPORTANT]
> `{"name":"k8s.io"}` deploys the bundled credential provider with an empty,
> default-deny namespace policy. Before an actor can have a credential
> injected, grant its atespace the namespaces whose Secrets it may read; see
> [Enable it](docs/guides/egress-credential-injection.md#enable-it).

3. In a **separate terminal**, send an HTTP request to increment the counter:
```shell
curl -X POST -i http://localhost:8000/
```

### Custom Setup and Deployment

You can deploy or cleanup specific Agent Substrate components using
the installation script. See `go run ./cmd/ate-setup` for all options.
```bash
# Re-deploy only ate-apiserver of the ATE system
go run ./cmd/ate-setup deploy apiserver

# Delete everything (core system and all demos)
go run ./cmd/ate-setup delete all
```

### Tearing down local `kind` resources

If you need to delete the local `kind` cluster and its registry (if it was created by `hack/create-kind-cluster.sh`):

```bash
./hack/delete-kind-cluster.sh
```

## Demos

We provide several sample applications demonstrating Agent Substrate's capabilities:

1. **[Counter Demo](demos/counter/README.md)**: A stateful Go HTTP server demonstrating state preservation across suspends/resumes, and on-demand actor resumption and routing via the Substrate router.
2. **[Sandbox Demo (Antigravity)](demos/sandbox/README.md)**: A secure, sandboxed execution environment (running Alpine Linux) that allows arbitrary shell execution while preserving filesystem state across sessions.
3. **[Claude Code Multiplex](demos/claude-code-multiplex/README.md)**: Demonstrates oversubscribing physical hardware by multiplexing multiple coding agents onto a limited pool of workers.
4. **[Multi-Template](demos/multi-template/README.md)**: Two `ActorTemplate`s running different binaries share one `WorkerPool`, even though the templates live in different atespaces.
5. **[Request Parking](demos/parking/README.md)**: An oversubscribed pool where the router holds inbound requests until a worker frees up, instead of returning `503`.

### Documentation & Guides
* [Architecture](docs/concepts/architecture.md): How the control plane, node supervisor, and networking stack fit together.
* [API Configuration Guide](docs/reference/api-guide.md): Detailed reference for configuring WorkerPools, ActorTemplates, Secrets, and Volumes.
* [Full CLI Documentation](docs/reference/kubectl-ate.md): Installation and usage for `kubectl-ate`.
* [Glossary](docs/reference/glossary.md): Core terms (Actor, Atespace, ActorTemplate, WorkerPool, Worker, ate-api-server, atenet, atelet, ateworker) and how they relate.
* [Observability Guide](docs/guides/observability.md): Guide to actor logging, metrics, and distributed tracing.
* [Extending Substrate](docs/security/extensibility.md): How to augment or replace the built-in ingress and egress gateways.
* [Authentication Guide](docs/guides/authentication.md): Which credential people, pods and CI jobs should use with the Substrate API, and how to trust more JWT issuers.
* [Actor JWTs](docs/guides/actor-jwts.md): What an actor JWT contains, how relying parties verify it, and how to rotate the signing key.
* [Egress Credential Injection](docs/guides/egress-credential-injection.md): How `atenet-egress` injects credentials into approved outbound requests outside the sandbox.
* [Egress Traffic](docs/guides/egress-traffic.md): Which protocols an Actor may reach the outside world with and which are blocked.
* [Request Parking](docs/concepts/request-parking.md): How the router parks requests through transient worker-pool saturation.
* [Sandbox Asset Cache](docs/concepts/sandbox-asset-cache.md): How a node caches the sandbox binaries actors run on, decides which runtimes it needs, and evicts to a budget.
* [Security Guide](docs/security/security.md): Security overview, security principles, and threat modeling.
* [Benchmarking Guide](docs/guides/benchmarking.md): Load tests, monitoring stack, and the orchestrated benchmark harness.

## Tour

### Commands

* `cmd/ateapi`: The core control plane API server exposing gRPC endpoints to manage actor and worker lifecycles.
* `cmd/atelet`: A node-level DaemonSet that supervises physical worker pods, coordinates snapshotting, and manages state transfers.
* `cmd/atecontroller`: A Kubernetes controller that reconciles WorkerPool custom resources.
* `cmd/ate-idp-server`: Serves the OpenID discovery document and key set that relying parties use to verify actor JWTs.
* `cmd/atenet`: A combined networking controller providing Envoy routing and proxy sidecars.
* `cmd/ateworker-gvisor`: An interior-pod helper running inside sandboxed worker pods to execute `runsc` checkpoint and restore commands.
* `cmd/ateworker-microvm`: The microVM peer of `ateworker-gvisor`, running actors as cloud-hypervisor VMs.
* `cmd/podcertcontroller`: A "polyfill" that provides Pod Certificate signers that will eventually ship in upstream Kubernetes (with different names).
* `cmd/kubectl-ate`: A CLI tool for managing Agent Substrate resources. See its [README](docs/reference/kubectl-ate.md).
* `cmd/benchmarking`: Synthetic workloads used by the load tests, including `glutton`, which consumes RAM, disk, and file descriptors on demand.
* `demos/`: Sample applications demonstrating Agent Substrate capabilities.

---

Part of [my always-on agent stack](https://github.com/Amz34) · [Awesome Agent Infrastructure](https://github.com/Amz34/awesome-agent-infrastructure) (135 live-checked building blocks).
