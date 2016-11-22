---
date: "2016-11-22"
draft: false
weight: 30
title: "Lab 03 - Open vSwitch (OVS)"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective

The objective of this lab is very similar to the Linux Bridge lab, except we will use an Open vSwitch (OVS bridge).

Some of the commands are presented in a different order than the Linux Bridge lab.

### Procedure

0. Close any terminals, and Firefox windows you currently have open within the remote desktop session.

0. From your remote desktop, open a new terminal session, and move to the student home directory.

  `student@beachhead:/$` `cd`

0. We explored this in the last lab, but just to cover all our bases. In this lab, we'll be using a little code snippet we invented called the *a3diff function*. If you tail the .bashrc file, you'll see it at the end. Just something we want you to be aware of, as we'll be using it in this lab to compare the output of some iproute2 commands.

  `student@beachhead:~$` `tail .bashrc`

  ```
  function a3diff {
      wdiff -n $1 $2 | colordiff
  }
  ```
  >
  If you're not a programmer, no big deal. All you have to know is that this little code snippit will save us some keystrokes, and allow us to do two (2) things:
  >
  1) Look for within-line differences (`wdiff`)   
  >
  2) Display the results in color (`colordiff`)

0. While we eventually want to create a OVS Bridge with virtual interfaces, let's first collect information about our current configuration. First display the L2 information.

  `student@beachhead:~$` `ip link list`
  
0. Great, now display L3 information.

  `student@beachhead:~$` `ip addr show`

0. Finally, show the current state of the routing table.

  `student@beachhead:~$` `ip route show`

0.   
  
  `student@beachhead:~$` `sudo ovs-vsctl show`

  * `student@beachhead:~$` `ip link list > /tmp/ovs-link-init`
  * `student@beachhead:~$` `ip addr show > /tmp/ovs-addr-init`
  * `student@beachhead:~$` `ip route show > /tmp/ovs-route-init`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-init`

0. Create network namespaces

  * `student@beachhead:~$` `sudo ip netns add luigi`
  * `student@beachhead:~$` `sudo ip netns add toad`
  * `student@beachhead:~$` `ip netns show`
  * `student@beachhead:~$` `ls /var/run/netns`
  
0. Examine network namespaces

  * `student@beachhead:~$` `sudo ip netns exec luigi ip link`
  * `student@beachhead:~$` `sudo ip netns exec toad ip link`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-l-link-init`
  * `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-t-link-init`

0. Create an open vswitch bridge and examine the changes

  * `student@beachhead:~$` `sudo ovs-vsctl add-br yoshis-island`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-br`
  * `student@beachhead:~$` `a3diff /tmp/ovs-show-init /tmp/ovs-show-br`
  * `student@beachhead:~$` `ip link show > /tmp/ovs-link-br`
  * `student@beachhead:~$` `a3diff /tmp/ovs-link-init /tmp/ovs-link-br`
  * `student@beachhead:~$` `ip addr show > /tmp/ovs-addr-br`
  * `student@beachhead:~$` `a3diff /tmp/ovs-addr-init /tmp/ovs-addr-br`

0. Create virtual ethernet interfaces which will get added into the namespaces

  * `student@beachhead:~$` `sudo ip link add eth0-luigi type veth peer name veth-luigi`
  * `student@beachhead:~$` `sudo ip link add eth0-toad type veth peer name veth-toad`
  * `student@beachhead:~$` `ip link show > /tmp/ovs-link-veths`
  * `student@beachhead:~$` `a3diff /tmp/ovs-link-br /tmp/ovs-link-veths`

0. Add the virtual ethernet interfaces into the network namespaces 

  * `student@beachhead:~$` `sudo ip link set eth0-luigi netns luigi`
  * `student@beachhead:~$` `sudo ip link set eth0-toad netns toad`
  * `student@beachhead:~$` `ip link show > /tmp/ovs-link-vethmove`
  * `student@beachhead:~$` `a3diff /tmp/ovs-link-veths /tmp/ovs-link-vethmove`
  
0. Examine the changes inside the network namespaces
 
  * `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-l-link-veth`
  * `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-t-link-veth`
  * `student@beachhead:~$` `a3diff /tmp/ovs-l-link-init /tmp/ovs-link-l-link-veth`
  * `student@beachhead:~$` `a3diff /tmp/ovs-t-link-init /tmp/ovs-link-t-link-veth`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr > /tmp/ovs-l-addr-init`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr > /tmp/ovs-t-addr-init`

0. Add the namespace interfaces to the open vswitch bridge

  * `student@beachhead:~$` `sudo ovs-vsctl add-port yoshis-island veth-luigi`
  * `student@beachhead:~$` `sudo ovs-vsctl add-port yoshis-island veth-toad`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-veth`
  * `student@beachhead:~$` `a3diff /tmp/ovs-show-br /tmp/ovs-show-veth`

0. Bring all the interaces up and examine the changes

  * `student@beachhead:~$` `sudo ip link set veth-luigi up`
  * `student@beachhead:~$` `sudo ip link set veth-toad up`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip link set dev eth0-luigi up`
  * `student@beachhead:~$` `sudo ip netns exec toad ip link set dev eth0-toad up`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-l-link-up`
  * `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-t-link-up`
  * `student@beachhead:~$` `a3diff /tmp/ovs-l-link-veth /tmp/ovs-link-l-link-up`
  * `student@beachhead:~$` `a3diff /tmp/ovs-t-link-up /tmp/ovs-link-t-link-up`

0. Add ip addresses to the internal interaces

  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr add 172.16.2.100/24 dev eth0-luigi`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr add 172.16.2.101/24 dev eth0-toad`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr`
  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr > /tmp/ovs-l-addr-ip`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr > /tmp/ovs-t-addr-ip`
  * `student@beachhead:~$` `a3diff /tmp/ovs-l-addr-init /tmp/ovs-link-l-addr-ip`
  * `student@beachhead:~$` `a3diff /tmp/ovs-t-addr-init /tmp/ovs-link-t-addr-ip`

0. Why can't we reach the new addressess?

  * `student@beachhead:~$` `ping -c 2 172.16.2.100`
  * `student@beachhead:~$` `ping -c 2 172.16.2.101`
  * `student@beachhead:~$` `ip route`

0. These two IP's should be able to ping each other though

  * `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  * `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`

0. Change the vlan tags for the two interfaces and show they can no loger connect with each other

  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-luigi tag=100`
  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-toad tag=101`
  * `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  * `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-tags`
  * `student@beachhead:~$` `a3diff /tmp/ovs-show-veth /tmp/ovs-show-tags`

0. Cleanup 

  * `student@beachhead:~$` `sudo ip netns del luigi`
  * `student@beachhead:~$` `sudo ip netns del toad`
  * `student@beachhead:~$` `sudo ip link del veth-luigi`
  * `student@beachhead:~$` `sudo ip link del veth-toad`
  * `student@beachhead:~$` `ovs-vsctl del-br yoshis-island`

