# detections/

One file per simulated attack; see `template.md` for the required format.

All testing is performed against systems owned by the author, inside the isolated
VULNERABLE VLAN (10.10.30.0/24). No external systems are targeted.

Each detection writeup documents: the exact command used, what Suricata triggered,
what Wazuh showed, the defender response, and an honest "what I learned" section
that includes detection gaps if the attack was not caught.
