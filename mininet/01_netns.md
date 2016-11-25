# mininet namespaces

0. Enumerate the system without mininet running

  `student@beachhead:~$` `ip link list`
  `student@beachhead:~$` `ip addr show`
  `student@beachhead:~$` `ip route show`

0. Save files for future comparison

  `student@beachhead:~$` `ip link list > /tmp/mn-ip-link-init`  
  `student@beachhead:~$` `ip addr show > /tmp/mn-ip-addr-init`
  `student@beachhead:~$` `ip route show > /tmp/mn-ip-route-init`

0. Start a basic mininet topology

  `student@beachhead:~$` `sudo mn`
  `mininet>`

0. In a new terminal, examine the changes to your root network namespace

  `student@beachhead:~$` `ip link list`
  `student@beachhead:~$` `ip addr show`
  `student@beachhead:~$` `ip route show`
  `student@beachhead:~$` `ip link list > /tmp/mn-ip-link-basic`  
  `student@beachhead:~$` `ip addr show > /tmp/mn-ip-addr-basic`
  `student@beachhead:~$` `ip route show > /tmp/mn-ip-route-basic`

  > what changed?
  >
  > what didn't change?

0. Run netns

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