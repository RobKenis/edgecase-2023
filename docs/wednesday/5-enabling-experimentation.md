# Enabling Experimentation with Ephemeral Environments (Workshop)

[Link to GitHub](https://github.com/rio/enabling-experimentation-workshop) _This dude has his first name as his GitHub username, damn.._

## The problems

Local test environments can have the wrong OS or are not reproducible. Remote environment can be forgotten and then they cost a lot of money.

## Ephemeral environments

Built for fast setup, isolation, auditable by default, and promote reuse.

One way to achieve this is classic CI environments. Run some commands to setup the environment, very straightforward. But it takes a long.

The other way is the declarative version with GitOps. Can be hard to debug, but are very fast.

## What we are going to build

Create PR -> Create environment -> Run tests -> Close PR -> Remove environment

### How did he do it

- GitHub: Source code management
- ArgoCD: Does GitOps things - Watches code in GitHub and deploys to the cluster
- [Kyverno](https://kyverno.io/): Rules engine and enforcer

- [vcluster](https://www.vcluster.com/): Creates a virtual cluster for each PR
- Kubernetes


Each PR is picked up by Argo, creates a new cluster using vcluster, that cluster is added as a target to ArgoCD. Then ArgoCD takes the code in GitHub and deploys it to that cluster.

The [ArgoCD Pull Request Generator](https://argocd-applicationset.readthedocs.io/en/stable/Generators-Pull-Request/) is used to create a vcluster per PR. Nothing custom, this is supported
by default by ArgoCD.
