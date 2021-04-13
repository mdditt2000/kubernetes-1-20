# Kubernetes 1.20 and Container Ingress Controller Quick Start Guide with BIG-IP High Availability (HA) Configuration

This page is created to document K8S 1.20 with integration of CIS and BIGIP in a HA cluster 

# Note

Environment parameters

* K8S 1.20 - one master and two worker nodes
* CIS 2.4.0
* AS3: 3.26
* BIG-IP 15.1

# Kubernetes 1.20 Install

K8S is installed on RHEL 7.5 on ESXi

* ks8-1-20-master  
* ks8-1-20-node1
* ks8-1-20-node2

## Prerequisite
About configuring VXLAN tunnels on high availability BIG-IP device pairs

By default, the BIG-IP® system synchronizes all existing tunnel objects in its config sync operation. This operation requires that the local IP address of a tunnel be set to a floating self IP address. In a high availability (HA) configuration, any tunnel with a floating local IP address would be available only on the active device, which would prevent some features, such as health monitors, from using the tunnel on the standby device. To make a tunnel available on both the active and standby devices, you need to set the local IP address to a non-floating self IP address, which then requires that you exclude tunnels from the config sync operation. To disable the synchronization of tunnel objects, you can set a bigdb variable on both devices.

### Disabling config sync for tunnels
In certain cases, you might want to disable config sync behavior for tunnels, such as when you need to make VXLAN tunnels functional on all devices in a BIG-IP® device group configured for high availability. The tunnel config sync setting applies to all tunnels created on the BIG-IP device.
Important: Disable config sync on both the active and standby devices before you create any tunnels.

Log in to the tmsh command-line utility for the BIG-IP system. Determine whether the variable is already disabled, by typing this command.

    tmsh list sys db iptunnel.configsync value

Disable the variable.

    tmsh modify sys db iptunnel.configsync value disable

Save the configuration.

    tmsh save sys config

**Now you can create tunnels with non-floating local IP addresses on both the active and standby devices**



## Create CIS Controller, BIG-IP credentials and RBAC Authentication

Configuration options available in the CIS controller
```
          args: [
            # See the k8s-bigip-ctlr documentation for information about
            # all config options
            # https://clouddocs.f5.com/products/connectors/k8s-bigip-ctlr/latest
            "--bigip-username=$(BIGIP_USERNAME)",
            "--bigip-password=$(BIGIP_PASSWORD)",
            # Replace with the IP address or hostname of your BIG-IP device
            "--bigip-url=192.168.200.92",
            "--bigip-partition=k8s",
            "--namespace=default",
            "--pool-member-type=nodeport", ----- As per code it will process as nodeport
            # Logging level
            "--log-level=DEBUG",
            "--log-as3-response=true",
            # Self-signed cert
            "--insecure=true",
          ]
```
**Note:** As per code it will process as nodeport but the service is configured as type-loadbalance 

```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: f5-hello-world
  name: f5-hello-world
spec:
  ports:
  - name: f5-hello-world
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: f5-hello-world
  type: LoadBalancer
  ```

Please let me know if you require additional information