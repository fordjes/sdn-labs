0. Open Wireshark

  `sudo wireshark &`

0. Start a new capture on `any` with a capture filter `port 6653`

0. Start a mininet basic topology with connection to a 'remote' controller

  `sudo mn --controller=remote,ip=127.0.0.1,port=6653`

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

0. stop the capture.

0. Find the following items in the pcap

  0. Lots of 
  0. Items
  0. which stu
  0. will provide
  0. for discovery
  0. and learning

