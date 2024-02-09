---
title: "AWS Tips and tricks"
weight: 2
---

## Helpful tips when working with AWS console

### General
- Its often helpful to open each new service in its own tab.  That way, you can refer back to each service independently without losing the info on your screen.
  - This trick is also useful when working in multiple regions.  Just be careful you make a change in the appropriate region!
- Search/filter within a service
  - ----------
  - Add CONTENT UPDATE
  - ----------
- AWS Network Routing
  - AWS VPC is inherently a Software Defined Network (SDN Router)
  - VPC Route Tables are similar to static routes in traditional routing
    - Generally they are attached to a subnet impacting the destination routing decisions for that subnet only
    - Putting the pieces together, AWS routing decisions happen at every hop along the traffic path
      - Symmetrical paths are the most important rule for AWS routing
        - Traffic must follow the same path in the outbound and inbound direction 
    - Sometimes RT can be associated with services (like IGW) allowing special routing decisions in specific scenarios
  - Every VPC has a default RT which allows communication between all subnets in the VPC
    - Every newly created subnet is associated with this default RT
    - you cannot change the VPC CIDR route entry in the default RT
    - You can add more specific subnet routes to a RT to do things like FortiGate NGFW Inspection of traffic _BETWEEN Subnets_ in a VPC