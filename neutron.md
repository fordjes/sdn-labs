1. Check out a freshly booted OVS-based Neutron network
  `sudo ovs-vsctl show`

2. Build the enternal network. OVS environments call this "ext-net" rather than "provider-net" like we often see in Liniu Bridge Networks
  `neutron net-create --shared --provider:physical_network external --router:external --provider:network_type flat ext-net`
  
3. What changed?  
  `sudo ovs-vsctl show`
  
   INSERT POWERPOINT Step 1 of 7 Networking a Freshly Bootstrapped Neutron 

4. Create the external network subnet   
   `neutron subnet-create --name ext-subnet --allocation-pool start=172.16.2.50,end=172.16.2.250 --dns-nameserver 10.0.0.1 --gateway 172.16.2.1  ext-net 172.16.2.0/24`

5. What changed?   
   `sudo ovs-vsctl show`
   `ip netns`
   
   INSERT POWERPOINT Step 2 of 7 Networking a Freshly Bootstrapped Neutron

6. Create a tenant network   
   `neutron net-create demo-net`
   
7. What changed?   
   `sudo ovs-vsctl show`
   INSERT POWERPOINT Step 3 of 7 Networking a Freshly Bootstrapped Neutron
   
8. Create the tenant subnet 
   `neutron subnet-create --name demo-net_subnet --dns-nameserver 10.0.0.1 --gateway 192.168.30.1 demo-net 192.168.30.0/24`

9. What changed?
   `sudo ovs-vsctl show`
   `ip netns`
   
    INSERT POWERPOINT Step 4 of 7 Networking a Freshly Bootstrapped Neutron  

10. Create a router to interconnect demo-net and ext-net   
   `neutron router-create demorouter`
   
11. What changed?   
    `sudo ovs-vsctl show`
    
    INSERT POWERPOINT Step 5 of 7 Networking a Freshly Bootstrapped Neutron   
    
12. Connect the new router to demonet    
    `neutron router-interface-add demorouter demo-net_subnet`
    
13. What changed?
    `sudo ovs-vsctl show`
    `ip netns`

    INSERT POWERPOINT Step 6 of 7 Networking a Freshly Bootstrapped Neutron   
    
14. Connect the new router to ext-net
    `neutron router-gateway-set demorouter ext-net`

    
15. What changed?
    `sudo ovs-vsctl show`       

    INSERT POWERPOINT Step 7 of 7 Networking a Freshly Bootstrapped Neutron 

16. Load some images so we can boot an instance
    
    `wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img`
    
    `openstack image create "cirros" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --public`
    
     `wget https://cloud-images.ubuntu.com/releases/16.04/release/ubuntu-16.04-server-cloudimg-amd64-disk1.img`
     
     `openstack   image   create    "xenial" --file ubuntu-16.04-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public`
     
17. Create availability zone so we can force a machine to boot on a specific compute node


18. Boot a cirros machine on compute1
        
19. What changed on neutron (controller node)?

    `sudo ovs-vsctl show`
    `ip netns`
    
20 SSH into compute1

21. What changed on compute1?

22 Now SSH into compute2

23. What networking is in place here (not much)

    `sudo ovs-vsctl show`
    `ip netns`

24. Launch a machine on compute2

25. SSH into compute node 2 and check out what may have changed.
    `sudo ovs-vsctl show`
    `ip netns`

   
INSERT POWERPOINT Step 7 of 7 Networking a Freshly Bootstrapped Neutron 
