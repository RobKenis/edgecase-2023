# Kubernetes as a Cornerstone in your Edge

In 2022, a Gartner survey finds that 85% of platform leaders without automation will increase automation within the upcoming years.

## What is the edge

Edge is not a data center. The edge is very close to where the magic happens, your hardware can get dirty.

### Kubernetes vs Challenges at the Edge

Kubernetes is built for scaling, load balancing, fault tolerance, self healing. Edge has configuration drift, weird updates and upgrades, the classic (read old) software stack, so eventually things will break.
One more challenge is expertise. Applying Kubernetes requires expertise, applying anything in a factory requires another set of expertise. Combining these 2 is not easy. Shipping software engineers with the
hardware to install it, is a waste of time. Why cant the engineer build a feature that can be skipped to a thousand customers, but they're busy installing hardware at a single customer? If you want to upgrade,
this might mean that the machine goes down. Chich-fil-A has an upgrade window during the night, _I'm assuming the 'Muricans can go 8 hours without fried chicken._ When you're upgrading a wind turbine, the
maintenance window is way harder, and it costs money to shut down during an upgrade.

## A peek under the hood

There's an operator in Kubernetes. You give it a spec of something to manage and the operator makes sure it works and is in the desired state. The [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) is run on the control plane and makes sure the cluster is in the desired state. A controller is a continuous loop of something that happens, it checks current
state and does actions on that state. It's a never ending loop, this is called the _reconcile loop_. 

When you do `kubectl`, you pass your desired state to the control plane, then some controllers run to match the desired state to the current state and tada, magic!

The controller connects to the kube-api to do some magic. But why should we limit ourselves to only use the kubernetes API. It can define the desired state of something I want to manage and use a controller
to manage the resource. When you ship for example a RPi with Talos, but you only ship a router and a switch, you can write a controller to manage the router and the switch.

## Building a k8s operator

_hey I know some of these words_

1. Make a CRD for a Cisco router, the spec includes `softwareVersion`
2. Make operator that implement [OperatorSDK](https://sdk.operatorframework.io/)
3. Manage something that allows to be managed

## Custom operator vs workload

An operator is focussed on a single task. Ansible or Terraform do not do an explicit thing, they manage the entire stack. Keeping the operator to a single thing, keeps it smaller and helps with easier monitoring.
Using an operator leans on top of the _well oiled_ machine called Kubernetes. _Depends how you write your operator, it's just a loop._ 

### A little sprinkle of GitOps

It simplifies software development and applies version controler and CI/CD best practices. Running `kubectl` is not a scalable option. GitOps enables you to run monitoring on your GitOps engine and gives
an insight to the syncing of your desired and actual state, which ~limits~ removes configuration drift.
