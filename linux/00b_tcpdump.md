version checking:
`tcpdump -h`
 
running TCPdump:
`sudo tcpdump -D

Lets capture packets on all interfaces by using the any option:
`sudo tcpdump -i any`

Instead of letting TCP dump run until interuppted, you can use lower case c option:
`sudo tcpdump -i any -c 5`

Usually it is easier to work with IP addresses and port numbers instead of these names. The lower case n option will take care of this:
`sudo tcpdump -i any -c 5 -n`

Just the first 96 bites of every packet is captured:
`sudo tcpdump -i any -c 5 -n -s96`

`clear`

For an example, lets generate some traffic over the eth1 interface:
`sudo tcpdump -i eth1 -n -t`

The next fields are the sequence and acknowledgement numbers. TCPDUMP by default uses relative numbers here. Lets look at an example:
`sudo tcpdump -i any -c20 -n tcp and dst port 49952 -t`

Relative sequence numbers can be turned off with the uppercase S option
`sudo tcpdump -i any -c20 -n tcp and port 49952 -i - S`

Lets look at sample output for DNS requests. Here is a capture on port 53:
`sudo tcpdump -i eth0 port 53 -n`

Now we'll do a wget to youtube.com to generate some dns traffic:
`wget youtube.com`

Often, tcpdump is used to save captures to a file for later analysis. This is done using the lower case w option:
`sudo tcpdump -i any -w capture.pcap`

When writing packets to a file, the lower case v option is very helpful:
`sudo tcpdump -i any -w capture.pcap -v`

A good way to limit the file size is to use the lower case c option:
`sudo tcpdump -i any -w capture.pcap -v c20`

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




