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
