# Scaling Wordpress to 1600 Donations per Minute

In today's age, it super important to bring personalized content. To achieve this, you need an environment where data flows free.

_Heeeeeeere comes the sales pitch_

...

_end of sales pitch_

## The Challenge

Users can search through a database of all knows songs, request a song and then donate some money to hear the song on the radio. The system had to be failure proof while handling parallel donations, peak of
1600 donations per minute. During peak moments, 70 donations per minute are made. Some maths later, this comes down to 25k euros per 10 minutes.

!!! note "Another challenge is that the dude likes talking and not getting to the point"

The actual challenge is that the product is based on Wordpress.

## The Solution

AWS was chosen to build on top of [EKS](https://aws.amazon.com/eks/). The application needs to be stateless with shared storage. [Bedrock](https://roots.io/bedrock/) is used to manage configuration. This helps
with putting Wordpress in a container and replicating it across environments. Containers bring the advantage of scaling. Wordpress is not built for horizontal scaling, your still stuck with shared storage for
Wordpress to write storage and media files.

### The Technical Stack

[Longhorn](https://longhorn.io/) brings shared storage with good performance. _But_ due to the nature of Longhorn, it creates a single NFS server for all Wordpress pods. This is where the problems starts, that
NFS server is a single pod, which brings a single point of failure. Another options is [AWS EFS](https://aws.amazon.com/efs/), which is slower and more expensive, but high available, so they went with EFS because
they didn't need that much storage.

For the database, they went with AWS RDS, high available, multi-AZ, all the bells and whistles. _But_ they went with the default parameter group, so they ran into the limit of max connections.

The managed Kubernetes service is not as managed as they would have liked, because they needed to run `helm install cluster-autoscaler`. [Crossplane](https://www.crossplane.io/) is leveraged to provision
RDS, EFS and IAM roles. This helps with managing the entire stack of a customer in a single Helm Chart. To ensure that manifests are deployed in the right order, the `"helm.sh/hook-weight"` annotation is 
used.

#### Autoscaling

[Keda](https://keda.sh/) scales based on (e.g. Prometheus) metrics. Each Wordpress Pod is running with the apache-metrics-exporter, and then Keda can scale the pods based on request latency. 

!!! note "Not sure why they pulled Keda in, HorizontalPodAutoscaler does this by default"

## Improving startup time

Create a DaemonSet to pre-pull the image to all the nodes. And overprovision the nodes _(they always have +3 nodes)_. 

!!! quote "I like how you've worked around the problem so far that you can't even see it anymore. ~ Rob"

## Fallback

Run a simple nginx container with an error page as a default backend. This handles request when all Wordpress pods are down. The static HTML is also stored on S3 so they can fallback even when everything
is gone.

## What have we learned

- Start load testing early on. They used K6.
- Have some monitoring. They used Prometheus, Loki, Grafana.

### Things that I learned

- They should have rewritten the donation endpoint to some proper language and put the frontend on S3.
