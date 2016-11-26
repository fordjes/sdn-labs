---
date: "2016-11-25"
draft: false
weight: 200
title: "Lab xx - Using Wireshark to Capture OpenFlow v1.3 Traffic"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is

### Procedure

0. Take a moment to clean up your remote Desktop. For now, close all other terminal spaces or windows you might have open.

0. Open a new terminal, then *cd* to the home student directory

  `student@beachhead:/$` `cd`

0. Launch Wireshark.

  `student@beachhead:~$` `sudo wireshark &`

0. Select **Capture > Options**

0. In the *Wirshark - Capture Interfaces* screen that pops up, *left-click* on the **any** interface.

0. In the *Enter a capture filter ...* box, type `port 6653`.

0. Click the Start button.

0. Start a Mininet basic topology with connection to a 'remote' controller

  `student@beachhead:~$` `sudo mn --controller=remote,ip=127.0.0.1,port=6653`

0. Start a ryu controller with the below file saved as l2.py

	``` python
	from ryu.base import app_manager
	from ryu.controller import ofp_event
	from ryu.controller.handler import MAIN_DISPATCHER
	from ryu.controller.handler import set_ev_cls
	from ryu.ofproto import ofproto_v1_0

	class L2Switch(app_manager.RyuApp):
			OFP_VERSIONS = [ofproto_v1_0.OFP_VERSION]

			def __init__(self, *args, **kwargs):
					super(L2Switch, self).__init__(*args, **kwargs)

			@set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
			def packet_in_handler(self, ev):
					msg = ev.msg
					dp = msg.datapath
					ofp = dp.ofproto
					ofp_parser = dp.ofproto_parser

					actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD)]
					out = ofp_parser.OFPPacketOut(
							datapath=dp, buffer_id=msg.buffer_id, in_port=msg.in_port,
							actions=actions)
					dp.send_msg(out)
	```

  `ryu-manager l2.py`

0. Pingall

  `mininet>` `pingall`

0. Stop the Wireshark capture by *left-clicking* the red square *stop* button.

0. In your resulting capture, click around to find the following types of OpenFlow packets.

  0. Lots of 
  0. Items
  0. which stu
  0. will provide
  0. for discovery
  0. and learning
