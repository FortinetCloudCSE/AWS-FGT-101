---
title: "FortiGate Secured East/West"
weight: 4
---


## FortiGate Security VPC securing East/West Traffic
- Goal: Utilize the provisioned Security VPC and Transit gateway architecture to provide security for East/West (Inter-VPC) flows with FortiGate NGFW.
- Task: Create FortiGate Policy objects and rules allowing East/West traffic to Acme Corp resources.

![](image-fgcp-tgw.png)

#### Summarized Steps (click to expand each for details)

1. Create Dynamic Address Objects.

    {{% expand title = "Detailed Steps..." %}}

- **1.1:** In the FortiGate GUI, navigate to **Policy & Objects > Addresses**, and click **Create new**.
- **1.2:** Create an address object with the **settings shown below** and click **OK**.

{{% notice tip %}}
Dynamic address objects allows creating address objects based on resource metadata such as VPC ID, Auto Scale Group, EKS Cluster or Pod, and even Tag Name + Value pairs applied to the resource. FortiOS is using AWS API calls behind the scenes such as ec2:DescribeInstances, eks:ListClusters, eks:DescribeCluster, etc to find running resources to match based on metadata and pull their IP address information. This is done on a frequent basis to keep the dynamic address object up to date automatically.
{{% /notice %}}

Name | Type | Sub Type | SDN Connector | Address Type | Filter Value
---|---|---|---|---|---
ProdWebFrontend | Dynamic | Fabric Connector Address | aws-instance-role | Private | Tag.env=prod AND Tag.app-role=web AND Tag.app-tier=frontend
ProdApiBackend | Dynamic | Fabric Connector Address | aws-instance-role | Private | Tag.env=prod AND Tag.app-role=api AND Tag.app-tier=backend

![](image-t4-1.png)

![](image-t4-2.png)

    {{% /expand %}}

2. Create Firewall Policy permitting East/West Traffic.
    
    {{% expand title = "Detailed Steps..." %}}

- **2.1:** Navigate to **Policy & Objects > Firewall Policy** and click **Create new**.
- **2.2:** Create a new policy with the **settings shown below** and click **OK** to allow east west traffic from Spoke1-Instance1 to Spoke2-Instance1.

![](image-t4-3.png)

    {{% /expand %}}

3.  Verify connectivity from **Spoke1-Instance1**.

    {{% expand title = "Detailed Steps..." %}}

- **3.1:** Navigate to the **EC2 Console** and go to the **Instances page** (menu on the left).
- **3.2:** Find & Select the **Spoke1-Instance1** instance and click **Connect > EC2 serial console**. 
  - **Copy the instance ID** as this will be the username and click connect. 
- **3.3:** Login to the EC2 instance:
  - username: `<<copied Instance ID from above>>`
  - Password: **`FORTInet123!`**
- **3.4:** Run the commands **`ping -c5 10.2.20.10`** and **`curl 10.2.20.10`** to connect to private resources, successfully.
- **3.5:** Run the command **`ssh 10.2.20.10`** to be blocked by firewall policy.
 
    {{% /expand %}}

4. Let's dig deeper to understand how all of this works.

    {{% expand title = "Detailed Steps..." %}}

- **4.1:** Navigate to **Log & Report > Forward Traffic** and you should logs for the traffic you generated. 
- **4.2:** **Double click** a log entry to view the **Log Details**.

{{% notice info %}}
In the **Source and Destination sections** of the log, we see the no NAT is applied and traffic comes in and goes out port2 of the Primary FortiGate. This is because of the **VPC routes in the all VPCs (Spoke1, NGFW, and Spoke2) are working together with the Transit Gateway (TGW) and Transit Gateway route tables to route** the east/west traffic through the primary FortiGate. This is a [**centralized design**](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-appliance-scenario.html) that is also commonly called an appliance, inspection, or security VPC.

We can also see denied traffic that is matching the **Implicit Deny** firewall policy.  With adding granular firewall policies & objects including dynamic address objects and security profiles, you can securely control traffic as desired.
{{% /notice %}}

    {{% /expand %}}

**This concludes this task**
