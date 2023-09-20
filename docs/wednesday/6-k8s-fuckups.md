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
