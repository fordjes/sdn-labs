## Virtual Interfaces

## TODO:
- Took Hilary about 5 minutes
- sudo required password

0. Create a pair of virtual ethernet interfaces 

  * `student@beachhead:~$` `sudo ip link add veth0 type veth peer name veth1`
  * `student@beachhead:~$` `ip link list`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  6: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
  7: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
  ```  

0. Bring up and add ip addresses to both virtual ethernet interfaces we created

  * `student@beachhead:~$` `sudo ip link set dev veth0 up`
  * `student@beachhead:~$` `sudo ip link set dev veth1 up`
  * `student@beachhead:~$` `ip link list`
  
    ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  6: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
  7: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
  ```
  
  * `student@beachhead:~$` `sudo ip addr add 10.0.0.1/24 dev veth0`
  * `student@beachhead:~$` `sudo ip addr add 10.0.0.2/24 dev veth1`
  * `student@beachhead:~$` `ip link list`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  6: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
  7: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
  ```

0. Examine the results
  
  > What are the mac addresses of our new interfaces?  
  > Where did these mac addresses come from?
  > How can we set these addresses if we wanted to ?

  * `student@beachhead:~$` `ip addr show veth0`
  
  ```
  7: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 scope global veth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7483:83ff:fe20:f14b/64 scope link 
       valid_lft forever preferred_lft forever
  ```
  
  * `student@beachhead:~$` `ip addr show veth1`
  
  ```
  6: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.2/24 scope global veth1
       valid_lft forever preferred_lft forever
    inet6 fe80::78a9:ffff:fedd:596/64 scope link 
       valid_lft forever preferred_lft forever
  ``` 

0. Ping from one veth interface to the other

  > Why wont this work?

  * `student@beachhead:~$` `ping -I veth1 10.0.0.1`
  
  ```
  No command 'pin' found, but there are 16 similar ones
pin: command not found
  ```

  * `student@beachhead:~$` `ping -I veth0 10.0.0.2`
  
  ```
  PING 10.0.0.2 (10.0.0.2) from 10.0.0.1 veth0: 56(84) bytes of data.
From 10.0.0.1 icmp_seq=1 Destination Host Unreachable
From 10.0.0.1 icmp_seq=2 Destination Host Unreachable
From 10.0.0.1 icmp_seq=3 Destination Host Unreachable
From 10.0.0.1 icmp_seq=4 Destination Host Unreachable
From 10.0.0.1 icmp_seq=5 Destination Host Unreachable
From 10.0.0.1 icmp_seq=6 Destination Host Unreachable
^C
  ```

0. Examine the routing table and run tcpdump whit the previous commands

  > When we force the ICMP packet destine for 10.0.0.2 on veth0 

  * `student@beachhead:~$` `ping -I veth1 10.0.0.1 > /dev/null &` _ping in the background_
  
  ```
  [1] 7340
  ```

  * `student@beachhead:~$` `sudo tcpdump -i veth1` _Ctrl^C to exit_
  
  ```
  requires sudo password
  ```
  
  * `student@beachhead:~$` `fg` _bring the ping back into the foreground and press Ctrl^C to quit_
  
  ```
  ping -I veth1 10.0.0.1 > /dev/null
  ```
  
  * `student@beachhead:~$` `ip route`
  
  ```
  default via 172.16.1.1 dev ens3 
  10.0.0.0/24 dev veth0  proto kernel  scope link  src 10.0.0.1 
  10.0.0.0/24 dev veth1  proto kernel  scope link  src 10.0.0.2 
  169.254.169.254 via 172.16.1.1 dev ens3 
  172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4 
  ```
  
  * `student@beachhead:~$` `ip route del 10.0.0.0/24 dev veth0`
  
  ```
  RTNETLINK answers: Operation not permitted
  ```
  
  * `student@beachhead:~$` `ip route del 10.0.0.0/24 dev veth1`
  
  ```
  RTNETLINK answers: Operation not permitted
  ```
  
  * `student@beachhead:~$` `ip route add 10.0.0.0/24 via 10.0.0.1 dev veth0`
  
  ```
  RTNETLINK answers: Operation not permitted
  ```
  
  * `student@beachhead:~$` `ip route`
  
  ```
  default via 172.16.1.1 dev ens3 
  10.0.0.0/24 dev veth0  proto kernel  scope link  src 10.0.0.1 
  10.0.0.0/24 dev veth1  proto kernel  scope link  src 10.0.0.2 
  169.254.169.254 via 172.16.1.1 dev ens3 
  172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4 
  ```

0. Remove all the virtual interfaces (back to scratch)

  > Why do we only need to run the delete once?

  * `student@beachhead:~$` `sudo ip link del veth0`
  
  * `student@beachhead:~$` `ip link list`
  
  ```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
  ```

