# Lights, Camera, Automation!

Objective: automate home without buying new hardware and without using commerical software.

## Initial seutp

K3s cluster in 2 locations, the cluster contains a master node, 3 worker nodes (2 are rpi). Everything for a working Grafana with InfluxDB.
Most of the applications are focused around energy monitoring and pool maintenance.

## Energy monitoring

In 2020, this dude got a smart reader for energy consumption. It connects to a P1 port, so pretty stable. At the time, nothing Dockerized existed, so a single RPI was used to only handle the P1 monitoring. 
After time evolved, a Docker image was published. When the Docker image was available, the application could be hosted on k3s. The pod needs to run on a specific node because it needs to access the P1 port 
and needs privileged access to the host to read the serial port. To get around that, this dude build a serial-to-network proxy which exposed the serial port over socat. Now the contianer can run anywhere and 
accesses the proxy on the specific node.

- https://github.com/intelwolf/p1monitor
- https://github.com/jeroenboot/p1monitor

## Garden lighting

Initially, a dectector was installed. When it becomes dark, send a signal, lights come on, _sick_! The downside is that the sensor is battery powered, which causes a lot of headache when the battery is low
or needs to be replaced. This also does not do timers, so the light is on the entire night and uses a lot of energy. Also..sensor degradation.

Intermediate solution is an on-off switch _lol_, which was later improved to a CronJob toggling the light on and off.

!!! note "This talk is the literal opposite of the previous talks. More like "how to make my home automation as complex as possible"

## Robot mower

Everything was fine until they got a dog that interfered with the mower. This caused the robot mower to fail almost every day.

The problem: how do we make the mower smart enough to keep working around the dog

Solution: a sonar system on the mower. The sonar sensor is connected to an RPi, and checks what the distance is between the mower and an object. This is used to calculate if the mower is home or not. Basically
`when distance > 25cm then mower is out of the house`. This information is sent to an Influx database and presented to a Grafana dashboard. The mower battery lasts 1h30. When the mower is out for longer than 90m and is not home, an alert is sent that the mower might be lost.

## Pool control

Problem: the pool is not very hygienic. 

The pump has 2 settings: heat and cool. When the water is hot enough, the pump stops running, but this also causes the filter to stop _lol_. So we need a timer again to make sure the filter is running
when the heater is off.

Solution: The CronJob reads the temperature on the roof and the water temperature every 5 minutes. This is used to determine if the pump needs to run. If the pump has not run for 24h, turn it on for some
time so the water is filtered, even when the water is hot enough. Everything is also stored in a database for statistics of course.

Another problem: heating system is too slow. The pipes go over the roof to heat the water using the sun, which makes the wife mad because she wants to swim _now_. So let's add another heating pump to 
heat the water quicker.

Another problem: the RPi powering this was not very stable. So we need to move the entire thing to a Kubernetes cluster. Make the RPi handle the communication to the pool hardware, expose it to
the cluster and do the compute in a cluster for HA. [Something like this, I guess](https://github.com/platformio/platformio-docs/blob/develop/plus/pio-remote.rst).

## Microservices and CD

Initial setup has a Dockerfile per service. This is annoying for updates. Now we have a single Dockerfile which is compiled for ARM, ARM64 and AMD64 so it can run on all nodes. Images are built by
GitHub, ArgoCD pulls the changes and deploys to Kubernetes.

## Security

Everything is publicly accessible, which poses some security questions. Ingresses are exposed through Traefik with Let's Encrypt. Some middleware is running with authentication. 
[Traefik ForwardAuth](https://doc.traefik.io/traefik/middlewares/http/forwardauth/). 

### Threat detection

How do I intercept signals that identity bad actors? [Falco](https://falco.org/) with [Sidekick](https://github.com/falcosecurity/falcosidekick).

Long story short: Falco intercepts everything using eBPF, uses a rule engine to intercept invading actions and triggers an action, for example an alert to Slack.
