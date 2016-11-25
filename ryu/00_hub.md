In this lab we will build a ryu-manager application (aka controller) step by step
For each commented section of the skeleton, each step will provide the nessisary code to be added.


0. Start with a base ryu application skeleton, save it as `hub.py`

``` python
from ryu.base import app_manager
from ryu.controller import ofp_event
from ryu.controller.handler import MAIN_DISPATCHER
from ryu.controller.handler import set_ev_cls
from ryu.ofproto import ofproto_v1_0

# create a class for our app manager
class L2Switch(app_manager.RyuApp):

    # Define the OpenFlow version this ryu app will use
    OFP_VERSIONS = [ofproto_v1_0.OFP_VERSION]

		# initialization incantation of this class (pass in args to self)
    def __init__(self, *args, **kwargs):
        super(L2Switch, self).__init__(*args, **kwargs)

    # decorator which tells Ryu when to call this function
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    # function to handle PacketIn
    def packet_in_handler(self, ev):

        # register values into shorter variable names
        msg = ev.msg
        dp = msg.datapath
        ofp = dp.ofproto
        ofp_parser = dp.ofproto_parser

        # create flood action
        actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD)]

        # create response to packetin
        out = ofp_parser.OFPPacketOut(
            datapath=dp, 
            buffer_id=msg.buffer_id, 
            in_port=msg.in_port,
            actions=actions
        )

        # actually send the packet to the requesting switch
        dp.send_msg(out)
```

0. Set the version number for the openflow protocol our controller will speak

    ```
    # Define the OpenFlow version this ryu app will use
    OFP_VERSIONS = [ofproto_v1_0.OFP_VERSION]
    ```

0. Setup the initialization of this controller class

    ```
		# initialization incantation of this class (pass in args to self)
    def __init__(self, *args, **kwargs):
        super(L2Switch, self).__init__(*args, **kwargs)
    ```

0.  decorator - tell ryu wen to call this function

    ```
    # decorator which tells Ryu when to call this function
    @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
    ```

0. packet handler

    ``` 
    # function to handle PacketIn
    def packet_in_handler(self, ev):
    ```

0. register values
 
    ```
        # register values into shorter variable names
        msg = ev.msg
        dp = msg.datapath
        ofp = dp.ofproto
        ofp_parser = dp.ofproto_parser
    ```

0. create action
 
    ```
        # create flood action
        actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD)]
    ```

0. create response 

    ```
        # create response to packetin
        out = ofp_parser.OFPPacketOut(
            datapath=dp, 
            buffer_id=msg.buffer_id, 
            in_port=msg.in_port,
            actions=actions
        )
    ```

0. send the packet 

    ```
        # actually send the packet to the requesting switch
        dp.send_msg(out)
    ```
