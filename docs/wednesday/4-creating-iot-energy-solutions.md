# Creating IoT Energy Solutions

https://vandebron.nl/

The idea is to exit grey energy and enter green energy. Currently, the Netherlands uses 40% green energy.

## Main challenges

We need to move from a handful of powerplants to a distributed system of people generating energy at home. This includes the migration from a dumb power grid to a smart grid with more production and
consumption.

Storing energy is way more imporant, since we're dependent on weather for green energy.

## Balancing the energy grid using IoT Edge devices

Primary use case: curtailment. The devices are used for cutting back the capacity of for example solar panels when the energy cannot be stored. This is a transitional solution until we have better
storage in place.

## Proof of Concept

Hardware: Intel NUC. Software: Ubuntu with k3s, basic auth and no update procedure.

They used [Talos Linux](https://www.talos.dev/), this is a minimal system, hardened by default. It is built for running Kubernetes. No SSH, only accessible over mTLS and has a readonly root filesystem.

Sometimes we want to reliably connect to the edge devices, for this they leverage a unique WireGuard tunnel per device.

## ONLOGIC

https://www.onlogic.com, https://www.jeffgeerling.com/tags/onlogic

Their vision is to become the global leader for hardware in Edge Computing.

70% of their hardware is in the industrial or rugged space. This means they have to handle abnormal circumstances regarding temperatures, salt air and weird power consumptions.

Their hardware is built partnering with RaspberryPi (but their supply chain is better _lol_).

## Extra

They use the [Modbus](https://www.modbustools.com/modbus.html) protocol wrapped in a WireGuard tunnel to communicate with the device.
