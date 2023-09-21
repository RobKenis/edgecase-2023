# Navigating the Stormy Seas of Kuberentes

_Brough to you by [StackState](https://www.stackstate.com/)._

## The problem

First you need a Kubernetes cluster, then you need Observability.

So we've created our cluster, deployed the frontend service, the user service, the order service, the catalogue service.. And then you start shipping and need to contact external services. Now we have a 
multi-cluster problem. 

## The Solution

A good graphical interface can help solve the skill gap.

### How StackState did it

They've deployed some [eBPF](https://ebpf.io/) magic in the cluster and hey ho, your telemetry is collected. They inject HTTP headers on your requests, this helps with distributed tracing without the need
for instrumenting it yourself. This also ensures that connections can be traced across cluster.

### What is eBPF

You write some low level code directly injected into the Linux Kernel.

Challenges: 512 bytes stack limit, 1 million instructions limit per eBPF program, finding the right probes (each kernel might have different hookpoints)
