---
date: "2016-11-23"
draft: false
weight: 220
title: "Lab xx - Mininet Namespaces - Learning about Linux Namespaces"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to explore how mininet uses namespaces, so that we might futher our understanding of the namespace concept.

### Procedure

0. Let's start off with some basic questions.

 - **Q1: What is a network namespace?**
   - A1: A network namespace is logically another copy of the network stack, with its own routes, firewall rules, and network devices.
 - **Q2: Can there be more than one (1) network namespace?**
   - A2: Yes. By default a process inherits its network namespace from its parent. Initially all the processes share the same default network namespace from the init process.
 - **Q3: How can I "see" the network namespaces within the system?**
   - A3: By convention a named network namespace is an object at */var/run/netns/NAME* that can be opened. The file descriptor resulting from opening */var/run/netns/NAME* refers to the specified network namespace. Holding that file descriptor open keeps the network namespace alive.
 - **Q4: What do I need to 'install' to work with network namespaces?**
   - A4: Nothing. Network namespaces entered the Linux Kerenl in 2.6.24 (~Jan 2009). From then it took about a year of futher development before they were actually usable.

0. Take a moment and clean up your remote desktop. Close any terminals, Wireshark windows, or Firefox browser sessions you might have open.

0. From your remote desktop, open a terminal session, and move to the student home directory.

  `student@beachhead:/$` `cd`

0. Enumerate the system **without** mininet running. First, display the current L2 configuration of the system.

  `student@beachhead:~$` `ip link list`

0. Now the L3 (IP) configuration of our current system with the *ip addr* command.

  `student@beachhead:~$` `ip addr show`

0. Finally, display the current state of routing.

  `student@beachhead:~$` `ip route show`

0. Save the output we just examined for future comparison. Write the output of *ip link list* to a new file called, *mn-ip-link-init*.

  `student@beachhead:~$` `ip link list > /tmp/mn-ip-link-init`

0. Write the output of *ip addr show* to a new file called, *mn-ip-addr-init*.

  `student@beachhead:~$` `ip addr show > /tmp/mn-ip-addr-init`

0. Write the output of *ip route show* to a new file called, *mn-ip-route-init*.

  `student@beachhead:~$` `ip route show > /tmp/mn-ip-route-init`

0. Start a basic mininet topology.

  `student@beachhead:~$` `sudo mn`
  
  ```
  *** No default OpenFlow controller found for default switch!
  *** Falling back to OVS Bridge
  *** Creating network
  *** Adding controller
  *** Adding hosts:
  h1 h2
  *** Adding switches:
  s1
  *** Adding links:
  (h1, s1) (h2, s1)
  *** Configuring hosts
  h1 h2
  *** Starting controller

  *** Starting 1 switches
  s1 ...
  *** Starting CLI:
  mininet>
  ```

0. Without closing the current terminal, open a new, second terminal.

0. Enumerate the system **with** mininet running. First, display the current L2 configuration of the system.

  `student@beachhead:~$` `ip link list`

0. Now the L3 (IP) configuration of our current system with the *ip addr* command.

  `student@beachhead:~$` `ip addr show`

0. Finally, display the current state of routing.

  `student@beachhead:~$` `ip route show`

0. Save the output we just examined for future comparison. Write the output of *ip link list* to a new file called, *mn-ip-link-basic*.

  `student@beachhead:~$` `ip link list > /tmp/mn-ip-link-init`

0. Write the output of *ip addr show* to a new file called, *mn-ip-addr-basic*.

  `student@beachhead:~$` `ip addr show > /tmp/mn-ip-addr-init`

0. Write the output of *ip route show* to a new file called, *mn-ip-route-basic*.

  `student@beachhead:~$` `ip route show > /tmp/mn-ip-route-basic`

0. Answer the following questions:

  - **Q1: What changed?**
    - A1:
  - **Q2: What stayed the same?**
    - Q2:
  - **Q3: Why does mininet create namespaces?**
    - Q3:

0. Run *ip netns list*. This command will reveal the 

  `student@beachhead:~$` `ip netns list`

  > Is this result suprising?  
  >
  > Do our veth's that mininet created look like they are connected to a network namespace?

0. Back in the mininet terminal enumerate the host's networking

  `mininet>` `h1 ip addr show`
  `mininet>` `h1 ip link list`
  `mininet>` `h2 ip addr show`
  `mininet>` `h2 ip link list`
  `mininet>` `s1 ip addr show`
  `mininet>` `s1 ip link list`

  > Does this increase our hunch that these are implemented with network namespaces?
  
  `mininet>` `exit`

0. Copy the below script into a file called `listns.py`

``` python
#!/usr/bin/python
#
# List all Namespaces (works for Ubuntu 12.04 and higher)
#
# (C) Ralf Trezeciak    2013-2014
#
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <http://www.gnu.org/licenses/>.
#
import os
import fnmatch
 
if os.geteuid() != 0:
    print "This script must be run as root\nBye"
    exit(1)
 
def getinode( pid , type):
    link = '/proc/' + pid + '/ns/' + type
    ret = ''
    try:
        ret = os.readlink( link )
    except OSError as e:
        ret = ''
        pass
    return ret
 
#
# get the running command
def getcmd( p ):
    try:
        cmd = open(os.path.join('/proc', p, 'cmdline'), 'rb').read()
        if cmd == '':
            cmd = open(os.path.join('/proc', p, 'comm'), 'rb').read()
        cmd = cmd.replace('\x00' , ' ')
        cmd = cmd.replace('\n' , ' ')
        return cmd
    except:
        return ''
#
# look for docker parents
def getpcmd( p ):
    try:
        f = '/proc/' + p + '/stat'
        arr = open( f, 'rb').read().split()
        cmd = getcmd( arr[3] )
        if cmd.startswith( '/usr/bin/docker' ):
            return 'docker'
    except:
        pass
    return ''
#
# get the namespaces of PID=1
# assumption: these are the namespaces supported by the system
#
nslist = os.listdir('/proc/1/ns/')
if len(nslist) == 0:
    print 'No Namespaces found for PID=1'
    exit(1)
#print nslist
#
# get the inodes used for PID=1
#
baseinode = []
for x in nslist:
    baseinode.append( getinode( '1' , x ) )
#print "Default namespaces: " , baseinode
err = 0
ns = []
ipnlist = []
#
# loop over the network namespaces created using "ip"
#
try:
    netns = os.listdir('/var/run/netns/')
    for p in netns:
        fd = os.open( '/var/run/netns/' + p, os.O_RDONLY )
        info = os.fstat(fd)
        os.close( fd)
        ns.append( '-- net:[' + str(info.st_ino) + '] created by ip netns add ' + p )
        ipnlist.append( 'net:[' + str(info.st_ino) + ']' )
except:
    # might fail if no network namespaces are existing
    pass
#
# walk through all pids and list diffs
#
pidlist = fnmatch.filter(os.listdir('/proc/'), '[0123456789]*')
#print pidlist
for p in pidlist:
    try:
        pnslist = os.listdir('/proc/' + p + '/ns/')
        for x in pnslist:
            i = getinode ( p , x )
            if i != '' and i not in baseinode:
                cmd = getcmd( p )
                pcmd = getpcmd( p )
                if pcmd != '':
                    cmd = '[' + pcmd + '] ' + cmd
                tag = ''
                if i in ipnlist:
                    tag='**' 
                ns.append( p + ' ' + i + tag + ' ' + cmd)
    except:
        # might happen if a pid is destroyed during list processing
        pass
#
# print the stuff
#
print '{0:>10}  {1:20}  {2}'.format('PID','Namespace','Thread/Command')
for e in ns:
    x = e.split( ' ' , 2 )
    print '{0:>10}  {1:20}  {2}'.format(x[0],x[1],x[2][:60])
#
```

  The above script was provided by [Open Cloud Blog](http://www.opencloudblog.com/?p=251)
  It helps us to enumerate "nameless" network namespaces which are only attached to a PID (`/proc/<PID>/ns`)

0. Run the `listns.py` script before and after re-setting up a basic mininet topology

  `student@beachhead:~$` `sudo python listns.py`
  `student@beachhead:~$` `sudo python listns.py > mn-ns-init`
  `student@beachhead:~$` `sudo mn`
  `mininet>`

  In a new terminal:

  `student@beachhead:~$` `sudo python listns.py > mn-ns-basic`

  > What changed?
  > 
  > How many namespaces did mininet create?
  >
  > What types are they?
  >
  > How many PID's are they associated with?
  >
  > What does a mininet namespace represent?

0. Connect to the mininet namespaces

  We can't connect to the namespaces with `ip netns exec [name]` because we dont have a name to connect to.
  We need to use nsenter wich can take a PID as an argument. 
  Run these commands for both PID's returned by `listns.py`

  `student@beachhead:~$` `sudo nsenter -t <NAMESPACE PID> -n ip addr show`
  `student@beachhead:~$` `sudo nsenter -t <NAMESPACE PID> -n ip link list`

0. Exit and cleanup

  `mininet>` `exit`
