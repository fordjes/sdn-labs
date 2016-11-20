# OVS Bridging 

This lab is very similar to the linux bridge lab except we will use an ovs bridge.
We will also demonstrate some of the commands in a diffrent order.

## a3diff

  Open .bashrc and read the funciton `a3diff`.  
  We will use this function in this lab to compare the output of some iproute2 commands.

  ```
  function a3diff {
      wdiff -n $1 $2 | colordiff
  }
  ```

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

0. Change the vlan tags for the two interfaces and show they can no loger connect with each other

  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-luigi tag=100`
  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-luigi tag=101`
  * `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  * `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-tags`
  * `student@beachhead:~$` `a3diff /tmp/ovs-show-veth /tmp/ovs-show-tags`
  * `student@beachhead:~$` ``
  * `student@beachhead:~$` ``



### Rocket Scientist Portion 

Let's setup dhcp for these two interfaces

0. Remve the current ip address configurations

  * `student@beachhead:~$` `sudo ip netns exec luigi ip addr del 172.16.2.100/24 dev eth0-luigi`
  * `student@beachhead:~$` `sudo ip netns exec toad ip addr del 172.16.2.101/24 dev eth0-toad`

0. Create new dhcp namespaces (for the dhcp servers)

  * `student@beachhead:~$` `ip netns add dhcp-luigi`
  * `student@beachhead:~$` `ip netns add dhcp-toad`

0. 
