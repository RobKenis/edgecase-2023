# The truth about Kubernetes

## Kubernetes in a datacenter

Rise of OpenShift and Rancher

## Edge computing

IoT can go from edge devices to Cloudflare edge locations. The landcape has
been evolving to support this, a great example is k3s. The biggest game changer
is GitOps. It allows us to have a single source of truth. Examples are Argo, Flux and
Fleet.

Talox Linux and Kairos make k8s part of the Linux operating system. Now we are in the 
situation were we can embed everything. "Why don't we flash k8s on top of a smart watch?"

## Kubernetes as an OS

Kubernetes is evolving as an OS instead of being on top of something.

## Lifecycle of Kubernetes

Started in 2014, getting pretty stable in 2023. A mature Kubernetes environment is an illusion.
Most companies are stuck around 2018, they are slowly moving to Kubernetes, tyring to understand
what is going on. They need help bridging the gap.

According to ChatGPT, complexity and the learning curve are the biggest hurdles in adopting Kubernetes.

### Learning curve

Kubernetes is 10% of the curve. There's also Observability, Security and Compliance, Advanced networking,
distributed storage, data management, software delivery (CICD), multi-cluster and multi-cloud management.
Get all this and we're production ready, easy.

> Complexity leads to operational overhead, which outweighs the benefits of running Kubernetes.

### OSS Observability Stack

Prometheus with Thanos, Grafana, PagerDuty for metrics. For logging, we need Grafana Loki. For tracing, we need
Grafana Tempo. So yeah..good luck learning this.

> Imaging being a system administrator and you know Nagios and you have to work with the new stack.

This is quite a big leap.

## Moving to Kubernetes

Go from barebones, to VMs, to containers, to Kubernetes. Some companies skip the containers step, so that
makes the leap even bigger. Going from barebones, to VMs to containers are all smalls steps. Going from containers
to Kubernetes is a big leap on its own.

### What can we do to help

- Babysteps: Do we need to build Skynet?!
  - How much of the 'production ready' stack do we need?
    - CNI and service meshes are simplicity killers.
  - Do we really need a management cluster?!
  - SaaS or Open Source?

> there is a joy in building the entire stack. But do companies really need it? Do we need something cool or
> do we need something efficient?

Don't build everything yourself, pick a SaaS provider or public cloud solution where possible.

> Less complexity leads to less time and less money!

## Conclusion

Reduce complexity and bridge the skill gap. Organize workshops during the implementation of Kubernetes.
