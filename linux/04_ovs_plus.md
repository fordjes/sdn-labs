---
date: "2016-11-22"
draft: false
weight: 30
title: "Lab 03b - ADVANCED - Open vSwitch (OVS)"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective

The objective of this lab is very similar to the Open vSwitch lab, however it takes a much deeper dive into OVS concepts. Only attempt this lab if you have:

  - 1) First finished the "BASIC" version of the OVS lab
  - 2) Have strong CLI skills

### Procedure

## OVS bridge, internal interfaces, and network namespace processes

0. Collect information about your initial configuration

  * `student@beachhead:~$` `ip link list`
  * `student@beachhead:~$` `ip addr show`
  * `student@beachhead:~$` `ip route show`
  * `student@beachhead:~$` `sudo ovs-vsctl show`

  * `student@beachhead:~$` `ip link list > /tmp/ovs2-link-init`
  * `student@beachhead:~$` `ip addr show > /tmp/ovs2-addr-init`
  * `student@beachhead:~$` `ip route show > /tmp/ovs2-route-init`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs2-show-init`

0. Basic Setup

  * `student@beachhead:~$` `sudo ip netns add peach`
  * `student@beachhead:~$` `sudo ip netns add bowser`
  * `student@beachhead:~$` `sudo ovs-vsctl add-br donut-plains`
  * `student@beachhead:~$` `sudo ip link add eth0-peach type veth peer name veth-peach`
  * `student@beachhead:~$` `sudo ip link add eth0-bowser type veth peer name veth-bowser`
  * `student@beachhead:~$` `sudo ip link set eth0-peach netns peach`
  * `student@beachhead:~$` `sudo ip link set eth0-bowser netns bowser`
  * `student@beachhead:~$` `sudo ovs-vsctl add-port donut-plains veth-peach`
  * `student@beachhead:~$` `sudo ovs-vsctl add-port donut-plains veth-bowser`
  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-peach tag=50`
  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-bowser tag=150`
  * `student@beachhead:~$` `sudo ip link set veth-peach up`
  * `student@beachhead:~$` `sudo ip link set veth-bowser up`
  * `student@beachhead:~$` `sudo ip netns exec peach ip link set dev eth0-peach up`
  * `student@beachhead:~$` `sudo ip netns exec bowser ip link set dev eth0-bowser up`
  * `student@beachhead:~$` `ip link list > /tmp/ovs2-link-config`
  * `student@beachhead:~$` `ip addr show > /tmp/ovs2-addr-config`
  * `student@beachhead:~$` `ip route show > /tmp/ovs2-route-config`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs2-show-config`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-link-init /tmp/ovs2-link-config`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-addr-init /tmp/ovs2-addr-config`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-route-init /tmp/ovs2-route-config`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-show-init /tmp/ovs2-show-config`

### Setup dhcp for these two interfaces

0. Create new dhcp namespaces (for the dhcp servers)

  * `student@beachhead:~$` `ip netns add dhcp-peach`
  * `student@beachhead:~$` `ip netns add dhcp-bowser`

0. Create internal ovs interfaces for the dhcp servers to run on

  * `student@beachhead:~$` `sudo ocs-vsctl add-port yoshi-island dhcp-peach -- set interface dhcp-peach type=internal`
  * `student@beachhead:~$` `sudo ocs-vsctl add-port yoshi-island dhcp-bowser -- set interface dhcp-bowser type=internal`
  * `student@beachhead:~$` `sudo ovs-vsctl set port dhcp-peach tag=50`
  * `student@beachhead:~$` `sudo ovs-vsctl set port dhcp-bowser tag=150`
  * `student@beachhead:~$` `sudo ip link set dhcp-peach netns dhcp-peach`
  * `student@beachhead:~$` `sudo ip link set dhcp-bowser netns dhcp-bowser`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-dhcp`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-show-config /tmp/ovs-show-dhcp`
  * `student@beachhead:~$` `ip link show > /tmp/ovs-link-dhcp`
  * `student@beachhead:~$` `a3diff /tmp/ovs-link-config /tmp/ovs-link-dhcp`

0. Bring up dhcp interfaces and assign ip addresses  

  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip link set dev dhcp-peach up`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip addr add 172.16.2.50/24 dev dhcp-peach`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip link set dev dhcp-bowser up`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip addr add 172.16.2.150/24 dev dhcp-bowser`

0. Start the dnsmasq service in both network name spaces

  
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach dnsmasq --interface=dhcp-peach --dhcp-range=172.16.2.51,172.16.2.149,255.255.255.0`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser dnsmasq --interface=dhcp-bowser --dhcp-range=172.16.2.151,172.16.2.249,255.255.255.0`

0. DHCPDISCOVER 

  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach dhclient eth0-peach`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip addr`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser dhclient eth0-bowser`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip addr`
 
0. OVS additional commands

  * `student@beachhead:~$` `sudo ovs-appctl show-commands`
  * `student@beachhead:~$` `sudo ovs-appctl fdb/show donut-plains`
  * `student@beachhead:~$` `sudo ovs-vsctl dump flows donut-plains`

0. Cleanup 

  * `student@beachhead:~$` `sudo ip netns del peach`
  * `student@beachhead:~$` `sudo ip netns del bowser`
  * `student@beachhead:~$` `sudo ip link del veth-peach`
  * `student@beachhead:~$` `sudo ip link del veth-bowser`
  * `student@beachhead:~$` `ovs-vsctl del-br donut-plains`
 
