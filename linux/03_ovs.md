# OVS Bridging 

##

This lab is very similar to the linux bridge lab except we will use an ovs bridge.
We will also demonstrate the commands in a diffrent order to emphizie that for the 
majority of the commands, order doesnt matter.

## Create a OVS Bridge with virtual interfaces


0. Collect information about your initial configuration

  * `student@beachhead:~$` `ip link list`
  * `student@beachhead:~$` `ip addr show`
  * `student@beachhead:~$` `ip rotute show`
  * `student@beachhead:~$` `sudo ovs-vsctl show`

  * `student@beachhead:~$` `ip link list > /tmp/ovs-link-init`
  * `student@beachhead:~$` `ip addr show > /tmp/ovs-addr-init`
  * `student@beachhead:~$` `ip rotute show > /tmp/ovs-route-init`
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
  * `student@beachhead:~$` `sudo ip link add eth0-toad netns toad`
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
  * `student@beachhead:~$` ``

0. Why can't we reach the new addressess?

  * `student@beachhead:~$` `ping -c 2 172.16.2.100`
  * `student@beachhead:~$` `ping -c 2 172.16.2.101`
  * `student@beachhead:~$` `ip route`

0. These two IP's should be able to ping each other though

  * `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  * `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`

### Rocket Scientist Portion 

Let's setup dhcp for these two interfaces

0. Remve the current ip address configurations

  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr del 172.16.2.100/24 dev eth0-luigi`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr del 172.16.2.101/24 dev eth0-toad`

0. Create new dhcp namespaces (for the dhcp servers)

  * `student@beachhead:~$` `ip netns add dhcp-luigi`
  * `student@beachhead:~$` `ip netns add dhcp-toad`

0. 
=========


0. Create a pair of virtual ethernet interfaces and bring them up

  * `ubuntu@ubuntu:~$` `ip link list`
  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ip link add veth1 type veth peer name veth2`
  * `ubuntu@ubuntu:~$` `sudo ip link set dev veth1 up`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`

0. Create a bridge, give it an IP,  and add the virtual interfaces to it

  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `ip address show > /tmp/addr-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `ip route show > /tmp/route-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl add-br br0`
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl list-br`
  * `ubuntu@ubuntu:~$` `sudo ip address add 10.0.0.1/24 dev br0`
  * `ubuntu@ubuntu:~$` `sudo ip link set dev br0 up`
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl list-ports br0`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/addr-before <(ip address show) | colordiff`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/route-before <(ip route show) | colordiff`

## Use the OVS bridge to access a network namespace 

0. Create a network namespace and move the 2nd veth into it 

  * `ubuntu@ubuntu:~$` `sudo ip netns add mario`
  * `ubuntu@ubuntu:~$` `sudo ip netns list`
  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ip link set veth2 netns mario`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`

0. Configure the namespace 
  
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ip addr add 10.0.0.2/24 dev veth2`
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ip link set dev veth2 up`

0. Demonstrate connectivity

  * `ubuntu@ubuntu:~$` `ping -c4 10.0.0.2`
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ping -c4 10.0.0.1`

## Cleanup

  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl del-br br0`
  * `ubuntu@ubuntu:~$` `sudo ip link del dev veth1`
  * `ubuntu@ubuntu:~$` `sudo ip netns del mario`
  * `ubuntu@ubuntu:~$` `ip link show` _expected veth1, veth2, br0 absent_
  * `ubuntu@ubuntu:~$` `ip netns list` _expected empty_





