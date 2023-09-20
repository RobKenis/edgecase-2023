# Kubernetes F*ckups

## 'Experience' can bite you

TLDR: Engineers 'with 20 years of experience' skip workshop sessions, but still want admin access.

```bash
# Will delete all namespaces lol, also kube-system
kubectl delete <name-of-namespace> --all
```

!!! note "Well this talk is going to be a rant.."

## One Char to Take Down a Cluster

Initial problem: Persistent Volume does not get attached. The Rancher cattle agent has some problems. The NodeGroup operator also went down. All these things were pointing to credentials,
managed by [kiam](https://github.com/uswitch/kiam). 

To troubleshoot this, let's look in Grafana. *Sike* Grafana and Loki are down. But they found out that 3 IPs are always coming up, these IPs were pointing to the default ingress of the 
cluster. All the pods could reach the IP address, but none of them could resolve the dns name. Someone deployed an ingress with `*.our.internal.domain`, this caused Route53 to resolve all
domains to that IP address. So `my-internal-service` does not resolve, because `my-internal-service.svc.cluster.local` does not resolve, but `my-internal-service.our.internal.domain` resolves.
The solution is to manually remove those DNS records from Route53, delete the faulty Ingress and wait for everything to work again. The structural solution was to create an
[Open Policy Agent](https://www.openpolicyagent.org/) rule to avoid wildcard records.

## Did I leave my keys in the house

Fun fact: Kubernetes exists of a bunch of smaller parts that talk to the k8s API. Pods are scheduled using the scheduler. It doesn't matter if the request comes from inside or outside the cluster.

When validating requests, we can have a ValidatingWebhook to validate if you are allowed to execute a request. You can make this webhook as strict or as lenient as you want by setting
`failurePolicy: Fail` or not.

If you deploy OpenPolicyAgent to validate, but OPA fails to start because it cannot validate itself...yeah good luck.

## It works on my dev cluster

_known as: kubectl-diff hidden features_

Someone manually adds resources limits that are not in the repository. Kubectl-diff will not show the difference and skip over the updates. So the resource limits are never removed.

!!! note "Note to self: I should probably stop manually fiddling around on staging clusters"

This works/breaks on any unspecified resource in your YAML: resources, labels (and extra labels), selectors, command, args..

> Just try it. It's fun. Until it isn't.

## Automatic cleanup

> When in doubt, nuke your cluster!

Problem: GitOps is really powerful. Really really powerful. GitOps will not double check, it will do what you tell it to do.

One of the options in ArgoCD is *pruning*. This means: whatever is not in Git, gets removed. *This means that typos become really fucking annoying*

## No images, please

Let's run Harbor in a cluster and then restart all nodes at once. Now Harbor is trying to start and pull the image from Harbor. _You get the idea._

How to fix: Move the Harbor image outside the cluster, like ECR, and don't try to do this again.

## Update a Cluster and OPA

What happens when old nodes are gone and new nodes enter? The `kubernetes-service` can no longer talk to the nodes where the Webhooks are running.

How to fix: `kubectl edit svc kubernetes` and update the endpoints.
