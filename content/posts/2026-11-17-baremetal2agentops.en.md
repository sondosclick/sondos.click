---
layout: post
title: "From Bare Metal to AI Agents: Sixty Years of Abstractions"
date: 2026-11-17
categories: [Infrastructure, AI, Technology]
tags: [Cloud, Kubernetes, GitOps, Observability, AgentOps, IaC, Control loops]
description: "How we went from swapping disks to declaring intent… without the disks ever going away."
featureimage: "/images/posts/2026-11-17-baremetal2agentops/baremetal2AI.png"
---

We're going to walk through several decades of systems administration in a single post. We'll start with a server you can touch and end with an AI proposing changes to production. My goal is to convince you that the two are far more closely related than they seem.

## The pattern that explains everything

At every stage the same thing happens:

> **Problem → Abstraction → New problem ↺**

We have a problem, we invent an abstraction, and we discover a new problem. Computing works surprisingly well this way. Remember this structure, because we'll watch it repeat with virtualization, cloud, containers, Kubernetes and AI.

## 1. Spoiler: the server still exists

There's a useful lie we keep repeating: *the cloud*. The cloud didn't make computers disappear. It made computers disappear from your immediate list of problems, which is very different.

Under a Pod there's a container, under that a VM, a hypervisor, a server, a rack, and beneath all of it, power and cooling. **Abstraction ≠ disappearance.**

What we call "the cloud" is essentially compute, network, storage, power and cooling with an API put in front of it: A/B power feeds, redundant networking, disks, sensors, fans, hot and cold aisles. *Serverless* doesn't mean there are no servers; it means the server is someone else's problem. Someone receives a heavy box, mounts it in a rack and connects power, network and management. When an SSD dies, someone ends up replacing it.

We're going to start by being that someone.

## 2. Bare metal: `web01` and the scale problem

First server: `web01`. One physical machine running Linux, nginx, our application, config files, local accounts, SSH and a monitoring agent. We set everything up by hand, and it works. *"It's done."* Famous last words.

Because the business has just asked for another one. And then another. How do we make `web02` exactly like `web01`? Do we copy the documentation? Clone a disk? Trust our memory? This is where the need for images, scripts and reproducible configuration is born.

## 3. Virtualization: one computer pretending to be many

The first big abstraction was virtualizing the hardware. A hypervisor abstracts CPU, memory, network and storage so several isolated operating systems can run on the same machine. We gained consolidation, snapshots and faster provisioning.

| Problem | Abstraction | New problem |
|---|---|---|
| Underused hardware | Virtualization | VM chaos, virtual networks, centralized storage |

As always: we solved one problem and created others. And virtualizing still doesn't spare us from buying, receiving and installing servers.

## 4. Cloud / IaaS: what if the server were an API?

The important change isn't moving servers far away. It's turning compute, network and storage into resources you request through an API. Before, there was a logistics process (order, box, rack, cables, installation) that took **weeks**. Now there's a call that takes **seconds or minutes**:

```yaml
POST /servers
cpu: 4
memory: 16 GB
disk: 100 GB
network: production
```

Let's take EC2 as an example, without turning this into an AWS tutorial. It's still a computer: server → *instance*, disk → *volume*, network → *VPC / subnet*, firewall → *security group*, image → *AMI*. **The interface changes; the fundamentals don't.** EC2 isn't the whole cloud: it's a virtual server managed through an API.

And an API lets us create 200 servers far too fast to configure them by hand. Cloud solved the problem of getting infrastructure and instantly turned manual configuration into a much bigger problem.

## 5. Automation: from *how* to *how it should be*

Automation doesn't start with a big platform. It starts when you get tired of typing the same command. If you do it twice, you should probably automate it. If you do it 200 times by hand, we need to talk.

### Imperative: the first superpower

```bash
# setup.sh
apt install -y nginx
echo "listen 8080;" >> app.conf
systemctl enable nginx
systemctl restart nginx
```

We tell the computer exactly which steps to run. It's simple and powerful, but it depends on the starting state. What happens if you run it twice?

```text
$ cat app.conf
listen 8080;
listen 8080;
$ systemctl restart nginx
ERROR: duplicate "listen" directive
nginx fails to start
```

### Declarative: tell me how it should be

```yaml
nginx:
  installed: true
  enabled: true
  port: 8080
  running: true
```

We move from describing actions to describing **intent**. I don't want you to run three commands: I want the package installed and the service running. Hold on to these two words: *desired state*.

### Different tools, different states

- **Ansible**: configures and orchestrates, usually agentless.
- **Puppet**: keeps configuration enforced continuously (agent + catalog).
- **Terraform**: provisions infrastructure (plan + state).

They overlap. The useful question isn't which one wins, but **which state you're trying to control** and which lifecycle you're managing.

### Everything as Code (and why)

Once we start describing things, we get carried away: application, infrastructure, configuration, policy, security, observability, dashboards, alerts, pipelines, runbooks. All as versionable artifacts.

What does the *as Code* suffix actually buy us?

- **Version**: every change has a history.
- **Review**: someone else looks at it.
- **Test**: it's tested before it's applied.
- **Reproduce**: same result every time.
- **Automate**: no manual steps.
- **Audit**: who changed what, and when.

Something being written in YAML doesn't automatically make it engineering. **The goal isn't to write more YAML; it's to do it with meaning and purpose.**

## 6. Containers: "it works on my machine"

Development has one library, production has another; the runtime changes; so does the configuration. What if we could package the application, its dependencies and its configuration reproducibly?

That doesn't create a small VM. A container is **an isolated process that shares the host's kernel**. Namespaces provide isolation; cgroups provide limits and accounting. An image is not a container, and a container is not an absolute security boundary.

## 7. Kubernetes: a control loop

Great, now we have 200 containers. Where do they run? How many are there? What happens if they die? How do they find each other? Where are the secrets? And the data? We need a system that is watching continuously.

Kubernetes is a platform for running distributed container systems, but the most useful way to see it is as a set of *control loops* that maintain desired state: **declare, observe, reconcile**.

From YAML to Pod, in six steps:

1. `kubectl apply` ("I want 3 replicas").
2. The **API server** receives and validates it.
3. **etcd** stores what's desired.
4. The **controllers** compare: are there 3? Some are missing.
5. The **scheduler** picks a node.
6. The node's **kubelet** starts and watches the Pods.

If the real state doesn't match the desired one, back to step 4. Don't memorize the boxes in the diagram: understand the mechanism.

A minimal example: we declare `spec.replicas: 3`, the system observes 2, and the controller doesn't remember a sequence of steps: it compares desired and observed level, and creates a replacement. Kubernetes doesn't run a script once: **it chases convergence**.

### Demo: let's kill a Pod

```text
$ kubectl get pods -w
pod-1   Running
pod-2   Running
pod-3   Running
$ kubectl delete pod pod-2
pod-4   Creating…
```

Kubernetes doesn't want that particular Pod. It wants **3 Pods**.

## 8. If everything is code… where does the truth live?

In production? In the documentation? In Git? In the CMDB? The word *truth* is dangerous; let's talk about **declared intent** instead.

Look at the *drift*:

| | Git | Production |
|---|---|---|
| replicas | 3 | 5 |
| version | 2.1 | 2.0 |
| region | eu-west | ??? |

Which one is right? We used to log into production to find out how production was configured. Now we want having to log in to discover the configuration to count as a process failure. Git isn't magically reality: it's the declarative source of intent that we've decided to govern.

**GitOps** adds a controller that watches both Git and the real environment and, if they diverge, tries to reconcile them. It doesn't necessarily wait for a pipeline to push commands using production credentials. Git expresses intent; the controller chases convergence.

## 9. Does this pattern ring a bell?

**Declare → Observe → Compare → Reconcile**, continuously. Kubernetes applies it to replicas; GitOps applies it to the state declared in Git. What other system declares, observes and corrects? Keep that blank in mind, because we'll come back to it.

First, let's go down beneath all of this. Where is that Pod really? Pod → container → Linux process → kernel → VM → hypervisor → physical server → rack → switch/fiber → power → cooling. **Kubernetes didn't abolish Linux. Cloud didn't abolish servers.** What changes is who has to think directly about each layer.

## 10. Observability: everything is green, users say it isn't

That sentence sums up roughly half of all interesting incidents: every local health check is green, but the end-to-end experience is broken. We need different signals that answer different questions:

- **Metrics**: what's happening?
- **Traces**: where?
- **Logs**: what happened?
- **Changes**: what did we just touch?

All correlated through a shared context (a trace ID, for example). It's a pedagogical simplification, but a useful one.

An example: a request takes 8 seconds. The frontend takes 120 ms, the API 180 ms, the database 90 ms… and *payments* 7.4 s. A metric tells you the scope and since when; the trace pinpoints *payments*; the logs give you the detail; the change history shows whether there was a recent deployment. **Observability isn't building dashboards: it's being able to ask questions you hadn't anticipated.**

## 11. Resilience: what if a region disappears?

With global traffic failing over from region A to region B, two questions appear:

- **RPO**: how much data can I afford to lose?
- **RTO**: how long does it take me to restore service?

Multi-region means traffic, data, identity and dependencies, not just copying two clusters. And it doesn't mean invulnerable: we gain availability and fault tolerance; we pay for it in cost, latency, replication, consistency and complexity. Two badly designed regions can give you twice the infrastructure and the same single point of failure. **Resilience comes with a bill, too.**

## 12. Enter AI

Now we have too many signals: metrics, logs, traces, Git, deployments, alerts, tickets, cloud, Kubernetes. That's *context overload*. AI starts to have an interesting use case, not because it magically knows how to operate, but because it can gather all that context faster than a person during an incident.

But correlating is not the same as having the authority to act. So, would you give `sudo` to an AI?

```text
$ sudo ai-agent █
```

The prudent answer is no. A probabilistic model shouldn't magically become our authorization system. The alternative is to enclose its proposals within deterministic controls: **AgentOps**.

> **Observe → Analyze → Propose → Policy → Approve? → Act → Verify**

Probabilistic reasoning **inside** deterministic controls: least privilege, limited blast radius, audit, rollback and a kill switch. If verification fails, roll back and go back to observing. Safe self-remediation isn't giving root to the model; it's enclosing a probabilistic proposal within deterministic controls.

And notice: AI follows the pattern from the start. The problem was too many signals; the abstraction, agents that correlate context and propose actions; the new problem, authority, plausible mistakes and auditability. **AI doesn't break the cycle: it closes it.** It was the blank left by the question marks.

## The full cycle

| Problem | Abstraction | New problem |
|---|---|---|
| Underused hardware | Virtualization | VM sprawl, network, storage |
| Servers that take weeks | Cloud / IaaS | 200 unconfigured servers |
| Manual configuration at scale | IaC | Apps that don't travel between environments |
| "It works on my machine" | Containers | 200 containers to orchestrate |
| Orchestrate and prevent drift | Kubernetes · GitOps | Too many signals |
| Context overload | AI · AgentOps | Authority, trust, auditability |

**Every abstraction opens the next cycle.**

## Closing

The story wasn't *cloud*. It was **control loops**: expressing intent, observing reality and shrinking the difference. IaC, Kubernetes, GitOps, autoscaling, self-healing, AgentOps… they aren't exactly the same mechanism, but they share an idea. The evolution wasn't simply toward the cloud: it was toward systems that manage state continuously.

And yet abstractions don't float; they rest on one another: AI agent, GitOps, Kubernetes, container, Linux, VM, hypervisor, server, rack. And beneath absolutely everything… there's a computer. It has a CPU, memory, disks and a network. It consumes electricity, generates heat and breaks. And somewhere, there's a person who ends up replacing that disk.

The tools you'll use ten years from now probably won't be these ones. But processes, memory, networking, storage, concurrency, consistency, security and debugging will still be there.

> The more abstractions you know, the higher up you can work. The more fundamentals you understand, the further down you can go when everything fails.

*Roque Rivera García, Technical Manager at NTT DATA.*
