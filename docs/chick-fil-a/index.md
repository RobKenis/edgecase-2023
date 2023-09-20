# Edge computing at Check-fil-A

How 3000 Kubernetes clusters helps our customers eat more chicken. 

## What is Chick-fil-A

Fast food that sells chicken. They do 60% of sales through drive through. They're looking at
building a location using 4 drive through lanes. Workers go up to the queue and takes orders.
Customers can order online a lot, this was the driver of building a 'mobile order drive thru'.
They have partnered with DoorDash (and others) to do the home delivery.

They are so busy that they can't keep enough food in physical locations. So they're supply chain
is super busy and efficient. Kinda dope stuff, it seems.

> Technology has the potential to enable business breakthroughs

## Edge Computing Architecture

Build things as close the action as required and no closer. They started in the cloud and moved to the
edge.

## What data is involved

IoT data: kitchen equipment, start cook time, end cook time..
App generated data: insights and forecasts, timers..
Point Of Sale data: keystrokes, user interaction..

### Sample use case

Data is collected from Lidar, POS and a team member's tablet. Data is sent to the edge (kubernetes) using MoscaJS (MQTT broker).
Data is processed and some is sent to the cloud. The forecase engine in the cloud calculates some stuff and sends it back to the 
edge application.

This helps worker and the kitchen to better prepare orders.

## Choosing Kubernetes

5 years ago they went to containers. If you want to run containers out of the cloud, there are number of possibilies
to run at the Edge. They went for k3s. Nomad, docker swarm, AWS outputs and normal Kuberentes didn't fit their needs,
mostly because it was too heavy to run.

K3s seemed like the right direction because it is single binary, easier to manage, sqlite. Less complexity was good. Their stack
looked like 3 Intel NUCs, Ubuntu and k3s. _I swear I didn't look at Chick-fil-A for my home lab lol_ Ubuntu 18.04 was chosen because
it was LTS at the time, they still get security updates and will not do full OS upgrades until hardware is replaced.

All nodes are on a different switch for HA on nodes and switches. 2 routers are installed for redudancy. Most of the edge devices
are connected through WiFi. Edge k3s is managed by the platform team, this provides an Auth stack, databases, deployment services..
The idea was to make the edge look like the cloud as much as possible for people using the services.

[OvelayFS](https://en.wikipedia.org/wiki/OverlayFS) is used for managing the systems.

## Teams

Teams are fairly autonomous and have the ability to make their own choices. This includes observability frameworks, own
technology and languages.

## Deployment with GitOps

> What's a good way to deploy a set of applications from different teams across 3000 clusters?

GitOps is a good way to describe the declarative state of the cluster and if it matches the
current state. Flux existed at the time, Argo wasn't really a thing. Chick-fil-A has a single git repository for
all restaurant locations. Teams can have a different release cadence. They have their own CI/CD pipeline. All their testing
has already been done. After that, they hit an API to deploy their applications.

The deployment orchrestrator can rollout deployment to selected restaurants. Depending on error rates, the rollout is
promoted to all restaurants or rolled back and feedback is given to the teams. This looks like [Argo Rollouts](https://argo-rollouts.readthedocs.io/en/stable/) 
but on a much much much larger scale.

## Persistence

> There are no guarantees. There is only best effort at the edge.

MongoDB is the primary data store. Using the database is more of a tool and not a long time storage mechanism. The data that teams are working
with at the edge is not imporant anymore after like 2 minutes, so it can be lost with almost no concern.

MQTT is used a lot, no a lot of REST/HTTP apis. All the traffic is routed to the cloud, process it there and send it back using MQTT.
[Vector](https://vector.dev/) is also used, but I missed why. 
When you need something important, send it up to the cloud, process it, send it back. *Important state is in the cloud*.

## Observability

Vector runs at the edge in the k3s cluster, fan-in all the metrics and logs. By default, Vector can filter things, so filter out
debug logs by default. When there's an issue and teams need more logging, you can enable it in Vector, Vector sends more logs and teams
get visibility. This matters because they are limited on bandwith, so Vector limits the data going out. Vector also collects metrics
and logs of IoT devices. For example a fryer is not always built by a reputable software company, so you want a layer between those logs and
your logging system in the cloud.

Vector in the cloud collects metrics and forwards them to the team's preferred solution like CloudWatch, Datadog..

## Principles

### Acknowledge constraints

Some constraints are good to build around and some you should just accept. Things like ports on a switch you should just accept. There's a small
physical space, no server closet, so you need to work in a confined space. Small budget leads to hardware constraints. Field tech support leads
to hard troubleshooting, so you want a hands-off solution.

### Just enough Kubernetes

Don't be cute, don't do too many things. At the edge, keep it lightweight. Keep it vanilla and as simple as possible, limit KubeAPI interactions.
Embrace the Kube movement and open source ecosystem. **Default to making things highly recoverable over super high available**. Things will break,
so make it easy to recover to a working state.

### Cattle, not pets

Zero touch provisioning minimized human touch, employees can just plug in the nodes and don't need to do anything else. A "wipe" pattern makes it possible
to wipe nodes and reprovision them to optimize troubleshooting. This also helps with the throw-away pattern, if a node doesn't work, throw it out and
replace it.

Re-hydrate pattern encourages teams to send critical data to the cloud. This makes it possible to rebuild the application after rebuilding the cluster.
It's like a new iPhone, you sync your new device with your Apple account and everything is back like it should be.

## Summary

Kubernetes is awesome at the edge. Acknowledge your constraints. Don't be cute and make things easy! Mirror the cloud paradigms as much as you can.

## Questions

- Do you do chaos engineering?
  - The platform team does this. For example "what happens when the network interface breaks during an OS upgrade". Updates are phased overtime. There's no
    formal chaos engineering where they break things in production.
  - More info
    - https://brianchambers.substack.com/
  - How do teams interact with other applications?
    - All through the standard MQTT protocol. When teams do testing, they have a physical 3 node environment, so they can mirror a real environment for testing.
  - How are nodes provisioned?
    - The process is pretty custom. Mostly OverlayFS and a custom implementation that looks like Ubuntu cloud-init. And somehting called "HAMs" but I can't find that.
  - What about distributed storage at the edge?
    - MongoDB is resilient across nodes in the cluster. There is no storage outside of the NUC, so everything is local. The MQTT broker runs on a single node, so the persistence in Mongo.
  - How big is the platform team?
    - Started of with 3 people. Went up to 6 when they were in production. Now there's 2 teams, one doing core platform and one doing IoT. But basically still a small team. The support
      work like wiping the nodes is external, so the plaform team does not have to do that.
  - How do you make the cluster HA?
    - K3s stores state in SQLite. If there's an issue on cluster-level, which doesn't happen a lot, they rebuild the cluster. _lol_ If a single node goes down, it's fine, there's a 
      process to recover from a master failure too. Then a replacement node is shipped.
  - How long does a store survive without upstream connectivity?
    - The entire stack does dark. A backup internet connection is reserved for incoming orders and credit card processing. The edge cluster is down, but the restaurant keeps functioning.
