### RYU REST

* `student@beachhead:~$` `ryu-manager simple_switch ofctl_rest --verbose`
* MININET topo start

need a lab for installing RESTClient Add-on to firefox, super easy to click through, hard to ansible-ize

GET /stats/switches
GET /stats/desc/[switch id]
GET /stats/flow/[switch id]

mn> pingall

GET /stats/flow/[switch id] - lots of flows now
POST /stats/flow/[switch id]
  Body: { "out_port" : 1 }

### Kill a bad actor with RYU REST

POST /stats/flowentry/add 
  Body: { "dpid" : 1, "cookie": 33, "priority": 33333, "match": {"in_port" : 3 }, "actions": [] }
POST /stats/flow/1
  Body: { "match": { "in_port": 3 } }

mn> pingall - expect h3 never pingable

### remove flows 

POST /stats/flowentry/delete_strict
  Body: { "dpid" : 1, "actions": [], "match": { "in_port" : 3 }, "cookie": 33,  "priority": 33333}
POST /stats/flow/1
  Body: { "match": { "in_port": 3 } }

mn> pingall - h3 returns to pingable
