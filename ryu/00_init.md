0. create a new file called base.py

``` python
from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import CONFIG_DISPATCHER, MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
from ryu.ofproto import ofproto_v1_3

# create a class for our app manager
class L2Switch(app_manager.RyuApp):

    # Define the OpenFlow version this ryu app will use
    OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]

    # initialization incantation of this class (pass in args to self)
    def __init__(self, *args, **kwargs):
        super(L2Switch, self).__init__(*args, **kwargs)

    # Handle switch feature interrogation event
    @set_ev_cls(ofp_event.EventOFPSwitchFeatures, CONFIG_DISPATCHER)
    def switch_features_handler(self, ev):
        return

    # Handle switch packet in event
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    def packet_in_handler(self, ev):
        return

```

0.  This is the bare minimum ryu-manager controller.  It actaully does nothing.

0. Startup wireshark and capture the packets which flow to/from the controller in this setup.

  Terminal1: `sudo wireshark &`
  
  Terminal2: `ryu-manager base.py`

  Terminal3: `sudo mn --controller=remote,ip=127.0.0.1,port=6653`

0. Answer the below questions about the wireshark capture:

  * What openflow packets were sent / received from the controller?
  * As a result of seeing these of packets, what does the ryu manager automatically do for us without any code?
  * If we ran `pingall` inside mininet would we see a PacketIn message to the controller?
  * Why or why not?

0. Tear down

  * `mininet> exit`
  * quit wireshark
  * Ctrl+C from ryu-manager
