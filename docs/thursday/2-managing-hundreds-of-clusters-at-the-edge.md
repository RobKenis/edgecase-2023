# Managing Hundreds of Clusters at the Edge

## Initial setup

- Rancher [K3OS](https://k3os.io/)
    - Problems: No custom driver support, no remote reboots, hard to change OS configuration, networking challenges, lack of support

## New setup

[Talos](https://www.talos.dev/) is built to be super fast and small. The image is 80MB or something, so there's plenty of room left for your own workloads. It's well suited for the edge, because
it is super hardened. Talos manages its own config using Kubernetes operators. The OS config is defined in a spec, it's picked up by the relevant controller and applied to the host.

It includes a feature called [Kubespan](https://www.talos.dev/v1.2/kubernetes-guides/network/kubespan/) which uses WireGuard to create a mesh network of different nodes.

### Managing Talos with Omni

[Omni](https://www.siderolabs.com/platform/saas-for-kubernetes/) is a management plane. It deals with authentication for clusters. They've built a feature to expose services running on Kubernetes to
be exposed over HTTP through Omni, so people with access to the control plane can access the services.

#### Talos V2

More flexible deployment options, customizable builds and better hardware support. Reduced hardware load, testing with V2 sees hardware running cooler.
