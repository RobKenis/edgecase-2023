# Pioneering GreenOps and Circularity

_Brought to you by [LeafCloud](https://www.leaf.cloud/)_

They wanted to do a normal cloud product with VMs and networks and Kubernetes. But do that without using extra energy. The way they do it is by using energy that is already being used for heating.
To get that to work, they had to think about what the infrastructure was going to run. The conclusion was that [OpenStack](https://www.openstack.org/) was the only way to compete with the other providers.
Side note: OpenStack is fucking hard to deploy.

The networking is quite different. Computers are placed in a basement from a building (e.g. appartment). This means that a datacenter is connected by a couple kilometers of fibre.

They run [Ceph](https://docs.ceph.com/en/latest/rados/) in a datacenter for storage. _I mean..that shit's on them._ The compute runs in appartment buildings, the output from the server cooler is used
to heat water in the appartment.

## Issues with OpenStack

Once it is deployed, it keeps running fairly well. There's a stability and a security challenge. [Masakari](https://wiki.openstack.org/wiki/Masakari) is used for failover of failing compute nodes.

!!! quote "Beinig sustainable is a plus for the user. But being stable is the main goal"

## The security problem

Software and security are a completely different stack. Getting certifications is about having processes in place and less about good software. _Can't believe I'm writing this down._ You have to check
that you have a couple hundres checks in place.
