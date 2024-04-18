---
title: "Single VPC & NATGW"
weight: 2
---


## **Private Outbound: Single VPC with NATGW** 
|                            |    |  
|----------------------------| ----
| **Goal**                   | Establish outbound only Internet access for EC2 Private instance.
| **Task**                   | Adjust the route table associations so ExPriv-Instance1 can reach the Internet.
| **Verify task completion** | Confirm outbound VPC connectivity from EC2 Instance via Ping to Internet from the EC2 instance

{{% notice warning %}} 
There are no security controls in this example. However, NATGW is a 1-way service, only allowing connectivity outbound.  While ExPriv-Instance2 can freely browse the Internet (outbound), no Internet hosts can connect to it (inbound).
{{% /notice %}}

![](../image-vpc-example.png)

#### Summarized Steps (click to expand each for details)

1. Login to **ExPriv-Instance2** and verify it cannot access the Internet.

    {{% expand title = "**Detailed Steps...**" %}}

- **1.1:** In the **EC2 Console** go to the **Instances page** (menu on the left) and select the **ExPriv-Instance2** instance.
- **1.2:** click **Connect > EC2 serial console**.
    - **Copy the instance ID** as this will be the username and click connect.
  ![](image-t1-4.png)
- **1.3:** Login to the EC2 instance:
    - username: <<copied Instance ID from above>>
    - Password: **`FORTInet123!`**
- **1.4:** Run the command **`ping -c5 8.8.8.8`** to connect to public resources.
  - This **SHOULD NOT** work at this point.

   {{% /expand %}}

2. Identify the relevant route table and add the proper route so 0.0.0.0/0 traffic is sent to **NATGW**.

    {{% expand title = "**Detailed Steps...**" %}}

- **2.1:** Navigate to the **VPC Console** and go to the **Subnets page** (menu on the left).
- **2.2:** Find the **Example-PrivateSubnet2** subnet.
- **2.3:** Select the **Route table tab** and click the actual route table name **rtb-.... / Example-PrivateRouteTable**.
  ![](image-t2-1.png)
- **2.4:** Select the **Example-PublicRouteTable** then select the **Routes tab** and click **Edit routes**.
  ![](image-t2-2.png)
- **2.5:** Add a default route (ie **0.0.0.0/0**) with a target of the **NAT Gateway** and click **Save changes**.
  ![](image-t2-3.png)

    {{% /expand %}}

3. Test Internet connectivity from **ExPriv-Instance2** again.

    {{% expand title = "**Detailed Steps...**" %}}

- **3.1:** Go back to the EC2 serial console and rerun the command **`ping -c5 8.8.8.8`** to connect to public resources successfully. 

   {{% /expand %}}

4. Let's dig deeper to understand how all of this works. 

    {{% expand title = "**Detailed Steps...**" %}}
	
- **4.1:** Run the command **`ifconfig ens5`** and take note of the instance IPv4 address. 
- **4.2:** Run the command **`curl ipinfo.io`**.

{{% notice info %}}
The instance has the private IP 10.0.20.10/24, but is and seen as coming from a public IP. This is because a [**NAT Gateway**](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) is providing outbound access to the internet for this private EC2 instance.
{{% /notice %}}

- **4.3:** In the **VPC Console** go to the **NAT gateways page** (menu on the left). 
- **4.4:** Find the **Example-NatGW** NAT Gateway and notice the public IP from the curl output matches the primary public IPv4 address assigned to NATGW. 
  - The NAT Gateway is deployed in the **Example-PublicSubnet2** subnet which has a route to the Internet through the [**AWS Internet Gateway (IGW)**](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html).
  - These AWS Networking components are allowing private outbound access to work successfully for this instance.

    ![](image-t2-4.png)

- **4.5** Below is a step by step of the packet handling for the outbound web traffic from ExPriv-Instance2.

Hop | Component | Description | Packet |
---|---|---|---|
1 | ExPriv-Instance2 -> 0.0.0.0/0 NAT GW | ExPriv-Instance2 sends outbound traffic to the VPC router (it's default gw) which routes the traffic to NAT GW as configured in the Example-PrivateRouteTable. | **<span style="color:black">10.0.2.10:src-port</span> -> <span style="color:blue">x.x.x.x:80</span>** |
2 | NAT GW -> 0.0.0.0/0 IGW | NAT GW changes the source IP to its own private IP and sends the traffic to VPC router. The VPC router routes traffic to IGW as configured in the Example-PublicRouteTable. | **<span style="color:purple">y.y.y.y:src-port</span> -> <span style="color:blue">x.x.x.x:80</span>** |
3 | IGW -> Internet | IGW changes the source IP to the associated EIP of NAT GW and routes the traffic to the internet. | **<span style="color:brown">z.z.z.z:src-port</span> -> <span style="color:blue">x.x.x.x:80</span>** |
4 | Internet -> IGW | IGW receives reply traffic, changes the source IP to the private IP of NAT GW, and sends the traffic to VPC router. The VPC router routes traffic to the NAT GW. | **<span style="color:blue">x.x.x.x:80</span> -> <span style="color:purple">y.y.y.y:dst-port</span>** |
5 | NAT GW -> ExPriv-Instance2 | NAT GW changes the source IP back to the private IP of ExPriv-Instance2 and routes the traffic to the VPC router which delivers the traffic to ExPriv-Instance2. | **<span style="color:blue">x.x.x.x:80</span> -> <span style="color:black">10.0.2.10:dst-port</span>** |

  ![](image-t2-5.png)

    {{% /expand %}}

### Discussion Points
- NAT GW providing many to 1 NAT, for outbound only.
- No internal reachability via NATGW, so no inbound probes seen on any of the private instances.
- General best practice, though thoroughly insufficient for overall security principles.
  
**This concludes this task**
