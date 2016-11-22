---
date: "2016-11-21"
draft: false
weight: 15
title: "Lab 01B - Linux Networking"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to demonstrate how to use the tcpdump tool, as this will be one of the many CLI tools we'll be working with in this class. Tcpdump prints out a description of the contents of packets on a network interface that match a boolean expression. The files saved by tcpdump are **\*.pcap files** which many users may recognize from working with Wireshark.

When tcpdump finishes capturing packets, it will report counts of:

  - *packets ``captured''* - The number of packets that tcpdump has received and processed
  - *packets ``received by filter''* - The meaning of this depends on the OS on which you're running tcpdump, and possibly on the way the OS was configured - if a filter was specified on the command line, on some OSes it counts packets regardless of whether they were matched by the filter expression and, even if they were matched by the filter expression, regardless of whether tcpdump has read and processed them yet, on other OSes it counts only packets that were matched by the filter expression regardless of whether tcpdump has read and processed them yet, and on other OSes it counts only packets that were matched by the filter expression and were processed by tcpdump)
  - *packets ``dropped by kernel''* - This is the number of packets that were dropped, due to a lack of buffer space, by the packet capture mechanism in the OS on which tcpdump is running, if the OS reports that information to applications; if not, it will be reported as 0.

### Procedure

0. Close any terminals you have open. Open a new terminal, and then change to the home directory.

  `student@beachhead:/$` `cd`

0. We can start by version checking the instance of *tcpdump* installed, as well as review the usage.

  `student@beachhead:~$` `tcpdump -h`
 
0. See the list of interfaces that *tcpdump* can listen on with the *-D* argument.

  `student@beachhead:~$` `sudo tcpdump -D`

0. Lets capture packets on all interfaces by using the any option:

  `student@beachhead:~$` `sudo tcpdump -i any`

0. Whoa! So much traffic. Press **CTRL + C** to quit.

0. Instead of letting TCP dump run until interuppted, you can use lower case c option to specify the number of packets you'd like to capture:

  `student@beachhead:~$` `sudo tcpdump -i any -c 5`

0. Usually it is easier to work with IP addresses and port numbers instead of these names. The lower case n option will take care of this:

  `student@beachhead:~$` `sudo tcpdump -i any -c 5 -n`

0. Let's augment the command further to just capture the first 96 bites of every packet is captured:

  `student@beachhead:~$` `sudo tcpdump -i any -c 5 -n -s96`

0. Let's clean up the screen, and go in a new direction.

  `student@beachhead:~$` `clear`

0. For an example, lets generate some traffic over the eth1 interface:

  `student@beachhead:~$` `sudo tcpdump -i eth1 -n -t`

0. The next fields are the sequence and acknowledgement numbers. TCPDUMP by default uses relative numbers here. Lets look at an example:

  `student@beachhead:~$` `sudo tcpdump -i any -c 20 -n tcp and dst port 22 -t`

0. Relative sequence numbers can be turned off with the uppercase S option

  `student@beachhead:~$` `sudo tcpdump -i any -c 20 -n tcp and dst port 22 -S`

0. Lets look at sample output for DNS requests. Here is a capture on all interfaces (-i any), port 53 (port 53), displaying IP addresses in lieu of names (-n), and capturing only 50 packets (-c 50):

  `student@beachhead:~$` `sudo tcpdump -i any port 53 -n -c 50`
  
0. It is more than likely that *tcpdump* will just sit as you're not making DNS requests. Within your remote desktop, click the Firefox icon.

0. Navigate to **www.google.com**, and search for **alta3 research**

0. By now tcp dump will have stopped capturing.
  
  > If you accidently started tcpdump to capture more than 50 packets, you can press **CTRL + C** to quit tcpdump.

0. Often, tcpdump is used to save captures to a file for later analysis. This is done using the lower case w option. The following command will capture on any interface (-i any), will writeout the capture to the file capture.pcap (-w capture.pcap), and capture a total of 5 packets (-c 5):

  `student@beachhead:~$` `sudo tcpdump -i any -w capture.pcap -c 5`

0. When writing packets to a file, the lower case v option (-v) runs a counter, which reflects the number of packets captured. The following command will capture on any interface (-i any), will writeout the capture to the file capture.pcap (-w capture.pcap), will display a packet capture counter (-v), and capture a total of 5 packets (-c 5):

  `student@beachhead:~$` `sudo tcpdump -i any -w capture.pcap -v -c 5`

0. We can limit a file size with the captial C option (-C). Before writing a raw packet to a savefile, tcpdump will check if file is currently larger than file_size and, if so, *close the current savefile and open a new one*. Savefiles after the first savefile will have the name specified with the -w flag, with a number after it, starting at 1 and continuing upward. The units of file_size are millions of bytes (1,000,000 bytes, not 1,048,576 bytes).

  `student@beachhead:~$` `sudo tcpdump -i any -w capture_10Mb.pcap -v -C 10`

0. The above command will go on indefinately, so press **CTRL + C** to quit.

Existing capture files can be read with the -r option:
`sudo tcpdump -n -r capture.pcap`

Packets go by too quickly on large files, so lets use a pipe and less, and that will help so that we can scroll up and down through the capture:
`sudo tcpdump -n -r capture.pcap | less

Filtering:
With this capture I'm using the host keyword to specify that I only want to capture traffic going to or from the IP at 10.0.0.3:
`sudo tcpdump -i eth1 -n host 10.0.0.3 -c5

If I now ping that vm, in the capture window immediately the icmp packets are seen:
`ping 10.0.0.3`

All the traffic not relating to 10.0.0.3 is being ignored for analysis. The source and destination keywords. This is the same capture as before except now you added the source keyword:
`sudo tcpfump -i eth1 -n src host 10.0.0.3 -c5`

Filter expression support the use of the logical operators 'and' and 'or'
`sudo tcpdump -i eth1 -n host 10.0.0.1 and host 10.0.0.3 -c5`

Filters can be used to isolate traffic to define TCP or UDP ports. Here, only port 80 traffic is captured:
`sudo tcpdump -i eth0 -n host 192.168.1.91 and port 80`

I'll do a wget to google.com and in the capture window, the HTTP traffic only on port 80 is seen:
`wget google.com`

Here is an example of a compound expression:
`sudo tcpdump -i eth0 -n "host 192.168.1.91 \
> and (port 80 or 443)"`

More filters can be used to include or ignore an entire subnet:
`sudo tcpdump -i eth0 -n -c100 "src net 192.168.0.0/16 \
> and not dst net 192.168.0.0/16 and not dst net 10.0.0.0/8"`

`ping google.com`
`nc google.com` 
`nc google.com 80`

Filters can be applied based on MAC addresses as well using the ether host keyword:
`sudo tcpdump -i eth0 ether host 28:16:2e:1f:25:49 -n -c10`

To see MAC addresses in captures, we use the lower case e option:
`sudo tcpdump -i eth0 ether host 28:16:2e:1f:25:49 -n -c10 -e`

More filters include protocol type. These can include the TCP, UDP, ICMP, ARP and RARP keywords:
`sudo tcpdump -i any ip 6`

Lets do a ping to an ipv6 ip, and in the capture window that icmp traffic shows up:
`ping6 2002::2`

Finally, here are some filters based on tcp flags:
`sudo tcpdump -i any "tcp[tcpflags \
> & tcp-syn !=0"`

`nc 10.0.0.3 80`
`nc 10.0.0.3 801`
`nc 10.0.0.3 802`

Similarly, here is an example to look for only tcp resets:
`sudo tcpdump -i any "tcp[tcpflags] \
& tcp-rst !=0"`

`nc 10.0.0.3 803`
`nc 10.0.0.3 804`

Adjusting tcp dumps output:
TCPDUMP has some options to adjust the output seen while looking at captured data:
`sudo tcpdump -i eth0 port 80 -c7 -XX`

`wget youtube.com`

Often the hex is not needed, so the -A option can be used instead:
`sudo tcpdump -i eth0 port 80 -c7 -A`

`wget youtube.com`

The -v option is used to provide more detail or verbosity in packet captures:
`sudo tcpdump -i eth1 -c15 -vvv`

`ssh 10.0.0.3`

This error can be ignored by using the uppercase K option:
`sudo tcpdump -i eth1 -c15 -vvv -K`

-q provides minimal quiet display output:
`sudo tcpdump -i eth1 -c15 -q`

`ssh 10.0.0.3`

Lastly we'll look at the timestamp display option which uses lowercase t:
`sudo tcpdump -i eth1 -c5 -q -t`

5 t's show the times since the first packet in the capture which is useful when measuring how long certain transactions take to complete:
`sudo tcpdump -i eth1 -c5 -q -ttttt`




