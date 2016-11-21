# Bridging 

## TODO:
- This lab took Hilary 20 minutes (one handed typing, just entering commands)

## What is a bridge
A bridge transparently relays traffic between multiple network interfaces.
A bridge connects two or more physical links together to form one bigger (logical) link.
A bridge does not know anything about protocols, it only sees Ethernet frames.
A bridge is essentially a switch, but with all the flexibility of Linux networking.

## a3diff

Open .bashrc and read the funciton `a3diff`.  
We will use this function in this lab to compare the output of some iproute2 commands.

```
function a3diff {
    wdiff -n $1 $2 | colordiff
}
```

We are saving ourselves some keystrokes in order to accomplish two things:

* look for within-line differences (`wdiff`)
* display the results in color (`colordiff`)

## Create a traditional Linux bridge with virtual interfaces

0. Collect information about your initial configuration

  * `student@beachhead:~$` `ip link list`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  ```
  
  * `student@beachhead:~$` `ip addr show`
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.4/24 brd 172.16.1.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe6f:4752/64 scope link 
       valid_lft forever preferred_lft forever
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    inet6 2003:db5:0:f102::1/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe2b:a795/64 scope link 
       valid_lft forever preferred_lft forever
  ```
  
  * `student@beachhead:~$` `ip route show`
  
  ```
  default via 172.16.1.1 dev ens3 
  169.254.169.254 via 172.16.1.1 dev ens3 
  172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4 
  ```
  
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-list-init`  
  * `student@beachhead:~$` `ip addr show > /tmp/ip-addr`
  * `student@beachhead:~$` `ip route show > /tmp/ip-route`

0. Create a pair of virtual ethernet interfaces 

  * `student@beachhead:~$` `ip link list`
    
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  ```
  
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-init` _save for comparison_
  * `student@beachhead:~$` `sudo ip link add veth1 type veth peer name veth2`ip 
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-veth1` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-link-list-init /tmp/ip-link-veth1 `
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    {+16: veth2@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
    {+    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff+}
    {+17: veth1@veth2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
    {+    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff+}
  ```

0. Bring up the first virtual ethernet interfaces we created

  * `student@beachhead:~$` `sudo ip link set dev veth1 up`
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-veth1up` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-link-veth1 /tmp/ip-link-veth1up `
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  16: veth2@veth1: [-<BROADCAST,MULTICAST,M-DOWN>-] {+<BROADCAST,MULTICAST>+} mtu 1500 qdisc noop state DOWN mode DEFAULT group default     qlen 1000
    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff
  17: veth1@veth2: [-<BROADCAST,MULTICAST,M-DOWN>-] {+<NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN>+} mtu 1500 qdisc [-noop-] {+noqueue+}       state [-DOWN-] {+LOWERLAYERDOWN+} mode DEFAULT group default qlen 1000
    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff
  ```

0. Create a bridge and add the virtual interfaces to it

  * `student@beachhead:~$` `sudo ip link add name br0 type bridge`
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-bridge` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-link-veth1up /tmp/ip-link-bridge`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  16: veth2@veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff
  17: veth1@veth2: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default         qlen 1000
    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff
    {+18: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
    {+    link/ether 06:37:83:bf:f2:f1 brd ff:ff:ff:ff:ff:ff+}
  ```
  
  * `student@beachhead:~$` `sudo ip link set dev veth1 master br0 `
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-bridge-veth1` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-link-bridge /tmp/ip-link-bridge-veth1`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  16: veth2@veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff
  17: veth1@veth2: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default         qlen 1000
    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff
    {+18: br0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
    {+    link/ether 06:37:83:bf:f2:f1 brd ff:ff:ff:ff:ff:ff+}
  ```

0. Add an ip address to the bridge interface

  * `student@beachhead:~$` `sudo ip address add 172.16.2.100/24 dev br0`
  * `student@beachhead:~$` `sudo ip link set dev br0 up`
  * `student@beachhead:~$` `ip address show > /tmp/ip-addr-bridge` _save for comparison_
  * `student@beachhead:~$` `ip route show > /tmp/ip-route-bridge` _save for comparison_
  * `student@beachhead:~$` `ip link show > /tmp/ip-link-list-bridge` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-route /tmp/ip-route-bridge `
  
  ```
  default via 172.16.1.1 dev ens3 
  169.254.169.254 via 172.16.1.1 dev ens3 
  172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4 
  {+172.16.2.0/24 dev br0  proto kernel  scope link  src 172.16.2.100 linkdown+} 
  ```
  
  * `student@beachhead:~$` `a3diff /tmp/ip-addr /tmp/ip-addr-bridge `
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.4/24 brd 172.16.1.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe6f:4752/64 scope link 
       valid_lft forever preferred_lft forever
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    inet6 2003:db5:0:f102::1/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe2b:a795/64 scope link 
       valid_lft forever preferred_lft forever
{+16: veth2@veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000+}
{+    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff+}
{+17: veth1@veth2: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue master br0 state LOWERLAYERDOWN group default qlen 1000+}
{+    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff+}
{+18: br0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000+}
{+    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff+}
{+    inet 172.16.2.100/24 scope global br0+}
{+       valid_lft forever preferred_lft forever+}
  ```
  
  * `student@beachhead:~$` `a3diff /tmp/ip-link-init /tmp/ip-link-list-bridge`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
{+16: veth2@veth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
{+    link/ether 66:22:9b:e1:f4:9b brd ff:ff:ff:ff:ff:ff+}
{+17: veth1@veth2: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue master br0 state LOWERLAYERDOWN mode DEFAULT group default qlen 1000+}
{+    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff+}
{+18: br0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000+}
{+    link/ether f2:2c:90:ca:99:c7 brd ff:ff:ff:ff:ff:ff+}
  ```
  
  * `student@beachhead:~$` `bridge link show br0`
  
  ```
  17: veth1 state LOWERLAYERDOWN @veth2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 master br0 state disabled priority 32 cost 2 
  ```

## Use the Linux Brige to access a network namespace 

0. Create a network namespace and move the 2nd veth into it 

  * `student@beachhead:~$` `sudo ip netns add mario`
  * `student@beachhead:~$` `ip netns list`
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-netns` _save for comparison_
  * `student@beachhead:~$` `sudo ip link set veth2 netns mario`
  * `student@beachhead:~$` `ip link list > /tmp/ip-link-netns-veth2` _save for comparison_
  * `student@beachhead:~$` `a3diff /tmp/ip-link-netns /tmp/ip-link-netns-veth2 `

0. Configure the namespace ip address
  
  * `student@beachhead:~$` `sudo ip netns exec mario ip link list`
  * `student@beachhead:~$` `sudo ip netns exec mario ip address show`
  * `student@beachhead:~$` `sudo ip netns exec mario ip link list > /tmp/mario-ip-link` _save for comparison_
  * `student@beachhead:~$` `sudo ip netns exec mario ip address show > /tmp/mario-ip-addr` _save for comparison_
  * `student@beachhead:~$` `sudo ip netns exec mario ip addr add 172.16.2.101/24 dev veth2`
  * `student@beachhead:~$` `sudo ip netns exec mario ip link set dev veth2 up`
  * `student@beachhead:~$` `sudo ip netns exec mario ip link list`
  * `student@beachhead:~$` `sudo ip netns exec mario ip address show`
  * `student@beachhead:~$` `sudo ip netns exec mario ip link list > /tmp/mario-ip-link-veth2` _save for comparison_
  * `student@beachhead:~$` `sudo ip netns exec mario ip address show > /tmp/mario-ip-addr-veth2` _save for comparison_

0. Demonstrate connectivity

  * `student@beachhead:~$` `ping -c4 172.16.2.101`
  * `student@beachhead:~$` `sudo ip netns exec mario ping -c4 172.16.2.100`

## Cleanup

  * `student@beachhead:~$` `sudo ip link del dev br0`
  * `student@beachhead:~$` `sudo ip link del dev veth1`
  * `student@beachhead:~$` `sudo ip netns del mario`
  * `student@beachhead:~$` `ip link show` _expected veth1, veth2, br0 absent_
  * `student@beachhead:~$` `ip netns list` _expected empty_





