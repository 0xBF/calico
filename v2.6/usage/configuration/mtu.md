---
title: Configuring MTU
canonical_url: 'https://docs.projectcalico.org/v3.0/usage/configuration/mtu'
---

Depending on the environment Calico is being deployed into it may be
helpful or even necessary to configure the MTU of the veth (or TAP) that is
attached to each workload and the tunnel devices if IP-in-IP is enabled.

### Selecting MTU size

Typically the MTU for your workload interfaces should match the network MTU.
If you need IP-in-IP then the MTU size for both the workload **and** tunnel
interfaces should be 20 bytes less than the network MTU for your network.
This is due to the extra 20 byte header that the tunnel will add to each
packet.

#### Common MTU sizes

| Network MTU | Calico MTU | Calico MTU with IP-in-IP | Calico MTU with VXLAN (IPv4) |
|-------------|------------|--------------------------|------------------------------|
| 1500 | 1500 | 1480 | 1450 |
| 9000 | 9000 | 8980 | 8950 |
| 1460 (GCE) | 1460 | 1440 | 1410 |
| 9001 (AWS Jumbo) | 9001 | 8981 | 8951 |

#### Default MTU sizes

The default MTU for workload interfaces is 1500, this is to match the most
common network MTU size.  The default MTU for the IP-in-IP tunnel device
is 1440 to match the value needed in GCE.

#### Using Flannel for Networking

When using Flannel for networking, the MTU for the network interfaces
should match the MTU of the flannel interface.  In the above table the 4th
column "Calico MTU with VXLAN" is the expected MTU when using Flannel
configured with VXLAN.

### Setting MTU for workload network interfaces

It is the job of the network plugin to create new interfaces, the current
major plugins are CNI and libnetwork. Currently Docker and the Mesos Docker
Containerizer integration use libnetwork.

CNI, which is used by Kubernetes and the Mesos Unified Containerizer, supports
configuring the MTU through the CNI configuration file.

The user will also want to configure Calico's IP-in-IP interface MTU when
IP-in-IP is enabled on the cluster. Refer to the MTU table at the top of the page
to choose the value that matches your environment.

> **Note**: The MTU on existing workloads will not be updated with these
changes.  To have all workloads use the new MTU, they must be restarted.
{: .alert .alert-info}

#### MTU configuration with libnetwork

The MTU of the veth pairs created by the Calico libnetwork plugin can be configured
by setting the `CALICO_LIBNETWORK_VETH_MTU` environment variable on the `calico-libnetwork`
process.

This should either be set on the standalone `calico-libnetwork` service, or on the
`calico/node` container as a whole.

#### MTU configuration with CNI

To set the MTU when using CNI the line `"mtu": <MTU size>` must be added to
the CNI configuration file.

Example CNI configuration

```json
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "mtu": 1480,
    "ipam": {
        "type": "calico-ipam"
    }
}
```


### Setting MTU for tunnel network interfaces

If IP-in-IP is enabled and the MTU needs to be modifed then this must be
configured by setting a Felix environment variable or using calicoctl to set
the proper configuration variable.  Felix will set the tunnel interfaces to
the specified MTU.

#### Setting tunnel MTU through Felix Environment variable

Passing in the environment variable `FELIX_IPINIPMTU` when running the
`calico/node` container will set the MTU for Felix to use.

#### Setting the tunnel MTU with calicoctl

To set the IP-in-IP MTU value for all calico nodes in your cluster, use the
following command to set the global config value.

```
calicoctl config set --raw=felix IpInIpMtu 1480
```

> **Note**: Setting the `IpInIpMtu` config option will result in an immediate
> update of the tunnel interface MTU on all of the active nodes in your cluster.
{: .alert .alert-info}


## Configuring MTU in Kubernetes self-hosted manifests

When using self-hosted manifests with Kubernetes, the MTU should be set by
updating the Calico manifest, applying the manifest with those changes, and
then restarting each of the calico-node pods.
