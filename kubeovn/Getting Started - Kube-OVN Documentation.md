

## Preparation

Kube-OVN is a network component that conforms to the CNI specification, and its operation needs to rely on the Kubernetes environment and the corresponding kernel network module. The following is the tested operating system and software version, environment configuration and required open port information.

## Software version

-   Kubernetes >= 1.16 and < 1.24, version 1.19 or later is recommended.
-   Docker >= 1.12.6, Containerd >= 1.3.4.
-   Operating system: CentOS 7/8, Ubuntu 16.04/18.04/20.04.
-   For other Linux distributions, you need to check whether the kernel module exists `geneve`, `openvswitch`and Kube-OVN depends on the above modules for normal work.`ip_tables``iptable_nat`

_Note_ :

1.  If the kernel version is 3.10.0-862, there is a bug in the kernel `netfilter`module which will cause the Kube-OVN built-in load balancer to fail to work, and the kernel needs to be upgraded. It is recommended to use the latest kernel of the official version of CentOS to ensure the security of the system. For related kernel bugs, refer to [Floating IPs broken after kernel upgrade to Centos/RHEL 7.5 - DNAT not working](https://bugs.launchpad.net/neutron/+bug/1776778) .
2.  Kernel 4.18.0-372.9.1.el8.x86\_64 of Rocky Linux 8.6 has a TCP communication problem [TCP connection failed in Rocky Linux 8.6](https://github.com/kubeovn/kube-ovn/issues/1647) , please upgrade the kernel to 4.18.0-372.13.1.el8\_6.x86\_64 or later.
3.  If the kernel version is 4.4, there is a problem with the corresponding kernel `openvswitch`module , it is recommended to upgrade or manually compile `openvswitch`the new version module for update
4.  Geneve tunnel establishment needs to check IPv6, which can be checked by `cat /proc/cmdline`checking the kernel startup parameters. For related kernel bugs, please refer to [Geneve tunnels don't work when ipv6 is disabled](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1794232) .

## Environment configuration[¶](https://kubeovn.github.io/docs/v1.10.x/start/prepare/#_3 "Permanent link")

-   Kernel startup needs to enable ipv6, if the kernel startup parameter contains it `ipv6.disable=1`needs to be set to 0.
-   `kube-proxy`It works normally, and Kube-OVN can be accessed through SVC IP `kube-apiserver`.
-   Confirm that the kubelet configuration parameter has enabled CNI, and the configuration is in the standard path. The following parameters should be included when the kubelet starts `--network-plugin=cni --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d`.
-   Confirm that other network plug-ins are not installed, or other network plug-ins have been cleared, and check that there are no other network plug-in configuration files in `/etc/cni/net.d/`the path . If you have installed other network plug-ins before, it is recommended to restart the machine after deleting to clean up residual network resources.

