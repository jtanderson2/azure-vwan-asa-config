# Cisco ASA Configuration - Connecting Resilient VPN to Azure VWAN Hub

This is largely based on [this](https://github.com/jwrightazure/lab/tree/master/asa-vpn-to-active-active-azurevpngw-ikev2-bgp) excellent article by Jeremey Wright. In order to better understand the design decisions and constraints that may steer you to towards this approach, I collaborated on a deep-dive session with Adam Stuart on the subject, where we delved into the options (and caveats!) around using Cisco ASAs to connect to Azure VWAN. Adam has written up the output of that [here](https://github.com/adstuart/azure-vwan-asa).

This repo provides a sanitised version of the working ASA configuration, with a few extra notes.

## Lab Topology

![](images/az-vwan-asa-vpn-diag.png)

## Configuration and Notes

ASA [config file](https://github.com/jtanderson2/azure-vwan-asa-config/blob/main/az-vwan-asa-config.txt) in the repo above. 

**Notes:**

* The On-Prem Publics IPs in this lab are NATTing on an external device to the ASA, though this approach would work the same with the public IPs directly on the ASA interfaces

* Static routes for the Azure BGP peer addresses are required over the Tunnel interfaces to allow the BGP neighborships to form. Interesting side note - the next-hop on these routes could be absolutely ANYTHING and this will still work, provided the correct Tunnel is selected as the interface. The ASAs force you to add a next-hop or the config won't be accepted, but in reality it's not required. I added the corresponding public IPs for some kind of sanity :)

* Although the VPNs themselves are Active/Active, the data path must be Active/Standby to avoid asymmetric routing over the tunnel interfaces (which the stateful ASAs don't like and would cause some traffic to be dropped!). I'm using as-path prepend in both directions to nail the Primary path as Tunnel1. This is the ONLY option to influence the inbound traffic back from Azure, however LocalPref or weight could instead be used to control the outbound flow from On-Prem.

> NOTE: It is possible to turn off TCP stateful inspection on the ASA for the Azure connection, which would allow asymmetric routing over the VPNs, but your security team probably won't thank you ;) If you still want to do it, read about TCP state bypass [here](https://www.cisco.com/c/en/us/support/docs/security/asa-5500-x-series-next-generation-firewalls/118995-configure-asa-00.html)


**Final Thought:**

Perhaps an ASA (or firewall in general) isn't the best device for this kind of solution, not least because it may not be possible to have 2 outside interfaces with distinct public IPs in all environments. If you have to use an ASA and can't provide 2 public IPs, check out the other solution offered up in the [article](https://github.com/adstuart/azure-vwan-asa) I linked to above.



