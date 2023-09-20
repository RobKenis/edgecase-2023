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
