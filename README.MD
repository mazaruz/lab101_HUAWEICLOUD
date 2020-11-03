# NET 101 - Build your first Virtual Private Cloud (VPC) on Huawei Cloud

### Objective
This workshop intends to help those who wanted to learn the foundation services on Huawei Cloud. It focuses on the network service called Virtual Private Cloud (VPC). Once you have finished the workshop you will understand how to create, manage and operate the network on Huawei Cloud. 

Our objective is to create a simple public/private network where there are two (2) ECSs running in both private and public subnet. Below are the requirements: 

- Private ECS **must not be able to reach from the internet** but we need to access it for downloading/updating software package
- Private ECS **must be able to** retrieve software package from the internet
- Public ECS must acting as Bastion or Jumphost and restrict the remote secure shell from **specific IP Address only**

## Before we begin
Please ensure that you have registered Huawei Cloud account and able to access the console with IAM user with necessary permission to create resources that we will be using in this workshop. 
> We DO NOT recommend the usage of root account or overly permissive IAM user for this workshop. 

## Lab 0 - Understanding Individual Account vs Enterprise Account

> Skip this lab if you are using "Individual Account"

## Overview
Huawei Cloud has two types of account; Invidual and Enterprise. The account type cannot be changed once you have specified the type. 

1. **Individual Account**
    This account type is for the customer that performing their own testing at home. It reduces the complexity of registration and verification process. Only credit card and phone number are required to get started. However, Real-name authentication is necessary if the customer needs to leverage resources from these following regions; CN North-Beijing4, CN East-Shanghai1, CN South-Guangzhou.

2. **Enterprise Account**
    Enterprise account is for the customer that performing their testing, R&D, or running production workload for their company. It requires Enterprise real-name verification which needs to be approved by Huawei before the customer can get started. This account type will allow the customer to leverage "Enterprise Center", an enterprise cloud integrated management service that provides cloud organization management and accounting management. The customer can create hierarchical organization and a master account, adding member accounts to the organization, or create enterprise projects based on the enterprise organization structure to manage resources, grant specific permissions to user group allowing them to operate or manage certain project independenly. 

### Objectives
1. Understand the differences between Individual Account and Enterprise Account
2. Learn how to create Enterprise Project (for Enterprise Account only)
3. Understand how Enterprise Project work (for Enterprise Account only)

## Duration 
5 minutes

### Task 1 - Create Enterprise Project
1. Login to Huawei Cloud Console, select **Enterprise** -> **Project Management** 
2. Click **Create Enterprise Project** to create new Enterprise Project under the organization you're in
    - Select **Test project** as project type.
> Project Type
>
> There are two Project Types when create Enterprise Project
> 1. Commercial Project: Resources of commercial projects are deployed in the production environment and are rated based on commercial discounts offered for the customer.
> 2. Test Project: Resources of test projects can be subscribed to using test coupons provided by the cloud service provider. The resources are rated based on prices on the cloud service provider website rather than commercial discounts for the customer. When all test coupons are used up, any subsequent fees generated will be deducted from your account balance.
    - Enter Project Name
    - Enter Description for The Enterprise Project
4. Once created, click on your project name to see more detail on the enterprise project
5. Click on **Permissions** 
    - Select your **User Group** assign your user group for this Enterprise Project
    - Select these following permission for this Lab 
        - ECS FullAccess
        - EVS FullAccess
        - VPC FullAccess
        - NAT FullAccess
6. Test the permission by navigate to VPC to see if there's no permission on the console

## Lab 1 - Introduction to Virtual Private Cloud (VPC)

## Overview
The Virtual Private Cloud (VPC) service enables you to provision logically isolated, configurable, and managable virtual networks for cloud servers, cloud containers, and cloud databases, improving cloud service security and simplifying network deployment. 

In this lab, we will be creating a single VPC with 2 subnets (1 Public Subnet and 1 Private Subnet) that span across two availability zones (AZs) which based on Best Practices from Huawei Cloud. The reference architecture as below picture

![Screenshot of the architecture](/images/architecture.png?raw=true)

### Objective
1. Understand the VPC components e.g. Subnet, NACL Route Table
2. Understand how subnets, route tables, and routing on Huawei Cloud work
3. Understand the differences between Public Network Access and Private Network Access
4. Able to design and create VPC to suit your own requirements
5. Able to create VPC Flow Logs to record traffics going in and out from VPC //Need to recheck the permission

## Duration
30 minutes

### Task 1 - Create a VPC and a private subnet with default route
1. Navigate to Huawei Cloud Console and select **VPC**
2. Click **Create VPC** on the top right corner
3. Enter the following information under **Basic Information** 
    - Region: Select the corresponding region
    - Name: Enter the VPC name e.g. *vpc-red*
    - CIDR Block: Enter the CIDR block that will be allocated for this VPC. Select the mask that you need. 
    > Please be mindful that when planning VPC CIDR, if VPC needs to communicate with each other or connect back to local on-premises, the CIDR range must not be conflicted
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
4. Enter the following information under **Default Subnet**
    - Name: Enter the subnet name e.g. *subnet-pri-red*
    - CIDR Block: Enter the CIDR block that is under the VPC CIDR range. Select the mask that you need
    - Leave the rest as default for now
5. Click **Create Now** at the bottom left of the screen
6. Navigate to **Network Console** and select **Virtual Private Cloud** to check if there is a new VPC created. To see more detail, click on the newly created VPC. There has to be 1 Subnet and 1 Route Table created as well.

> We recommended to change the default route table name of the newly created VPC to *rtb-pri-red* to avoid any confusion in the future

Now we have created our first VPC, subnet and route table. But the default route won't allow any access from the internet unless we configure an EIP, ELB or NAT Gateway with specific routes to route the incoming or outgoing traffic. 

> Default Route Table vs Custom Route Table
> 
> A Default Route Table is generated during VPC creation, once a subnet is created, it will associate this default route table with the subnet. Routes can be created, modified, or deleted but the default route table cannot be deleted. If VPN is created, the default route table automatically associates the route which cannot be deleted or modified. 
>
> A Custom Route Table can be created manually under specific VPC. The subnet then has to be associated with the custom route table to route the traffic. 

### Task 2 - Create a public subnet and a custom route table

1. Navigate to **Network Console** and select **Subnets**
2. Click **Create Subnet** on the top right corner
3. On the pop-up windows, enter the following information
    - VPC: Select the VPC that this subnet will be associated with
    - Name: Enter the name of the subnet e.g. *subnet-pub-red*
    - CIDR Block: Enter the CIDR block that is under the VPC CIDR range. Select the mask that you need 
    - Leave the rest as default. We will be associating a custom route later
4. Click **OK** to create the subnet
5. Click at the newly created subnet to see more detail

> When a subnet is created, there are five (5) reserved IP addresses that cannot be used. The reserved address are as the following;
> - **xxx.xxx.x.0 Network ID** This address is the beginning of the private IP address range and will not be assigned to any instance
> - **xxx.xxx.x.1 Gateway address** 
> - **xxx.xxx.x.253 Reserve for the system interface** This address is used by the VPC for external communication
> - **xxx.xxx.x.254 DHCP service address**
> - **xxx.xxx.x.255 Network broadcast address**
> These reserved addresses can be different. They depend on the customer's advance setting in their subnet configuration

6. Navigate to **Network Console** and select **Route Tables**
7. Click **Create Route Table** on the top right corner
8. On the pop-up windows, enter the following information
    - Name: Enter the name of the route table e.g. *rtb-pub-red*
    - VPC: Select the VPC that this route table will be relying on
    - Leave the rest as default, we will configure route settings after
9. Click **OK** to create the route table that we will be using for public access
10. On the pop-up windows, click **Associate Subnet** to associate this newly created route table with the public subnet that we created earlier
11. Click **Associate Subnet**, select the *subnet-pub-red*, and click **OK** 
12. On the warning pop-up windows, click **Yes**
13. Navigate to **Network Console** and select **Subnets**
14. You should see on the console that *subnet-pub-red* is associating with the route table name *rtb-pub-red*

### Task 3 - Create custom route to allow ingress and egress access via the subnets

To allow public network access, there are three (3) services that the customer can leverage as followed;

1. EIP (Elastic IP) - The EIP provides independent public IP Address and bandwidth for the internet access. EIPs can be bound to or unbound from Elastic Cloud Servers (ECSs), Bare Metal Server (BMSs), Virtual IP Address, NAT Gateway, or Load Balancer. There are various billing modes to meet service requirements
2. ELB (Elastic Load Balancer) - ELB distributes access traffic among multiple ECSs to balance application load, improving fault tolerance and expanding service capabilities of applications. 
3. NAT Gateway - NAT Gateway provides both Source NAT (SNAT) and Destination NAT (DNAT) for the resources in a VPC and allows servers to access or provide services accessible from the internet

In this lab, we will be performing two types of access (Ingress, Egress) while leveraging EIP and NAT Gateway to simulate the use case of public subnet and private subnet with a custom route table
- Ingress: We will be using EIP to attach the Elastic Cloud Server (ECS) that sitting in public subnet, and we will execute secure remote session to the ECS in private subnet
- Egress: We will be using NAT Gateway with Source NAT (SNAT) that sitting in public subnet to allow ECS in private subnet to access the internet 

Before we can configure our custom route table, EIP and NAT Gateway have to be created first. 

### Task 3.1 - Create EIPs for EC2
1. Navigate to **Network Console**, select **Elastic IP and Bandwidth**, and click **EIPs**
2. Click **Buy EIP** on the top right corner
3. There are various options to create EIP, in this lab, we recommended to create with these following setting for testing purpose:
    - Billing Mode: Pay-per-use
    - Region: Select the same region as your VPC
    - EIP Type: Dynamic BGP
    - Billed by: Bandwidth 
    - Bandwidth: 1 Mbps
    - Bandwidth Name: Enter the name for your bandwidth e.g. bandwidth-red-ec2
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
    - Quantity: 1
4. Click **Next**, the summary screen will show that you're purchasing EIP and Bandwidth together with the estimated price per hour. Click **Submit** to create the EIP.

### Task 3.2 - Create EIPs for NAT Gateway
1. Navigate to **Network Console**, select **Elastic IP and Bandwidth**, and click **EIPs**
2. Click **Buy EIP** on the top right corner
3. There are various options to create EIP, in this lab, we recommended to create with these following setting for testing purpose:
    - Billing Mode: Pay-per-use
    - Region: Select the same region as your VPC
    - EIP Type: Dynamic BGP
    - Billed by: Bandwidth 
    - Bandwidth: 1 Mbps
    - Bandwidth Name: Enter the name for your bandwidth e.g. bandwidth-red-nat
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
    - Quantity: 1
4. Click **Next**, the summary screen will show that you're purchasing EIP and Bandwidth together with the estimated price per hour. Click **Submit** to create the EIP.

### Task 3.3 - Create NAT Gateway with SNAT
1. Navigate to **Network Console** then click **Buy Public NAT Gateway** at the top right corner
2. Enter the following information:
    - Billing mode: Pay-per-use
    - Region: Select the same region as your VPC
    - Name: Enter the name for your NAT Gateway
    - VPC: Select the VPC that this NAT will be provisioned to
    - Subnet: Select the subnet that this NAT will be provisioned to. In this lab, we are choosing *subnet-pub-red* because we want to group internet facing components together. 
    - Type: Small
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
3. Click **Next**, the summary screen will show that you're purchasing NAT Gateway with the estimated price per hour. Click **Submit** to create NAT Gateway
4. On pop-up window, click **Add Rule** to add SNAT rule so that our NAT Gateway can route traffic from private ECS to the internet
5. At the SNAT Rules screen, click **Add SNAT Rule** 
6. Enter the following information
    - Scenario: VPC (We are creating this SNAT rule to route traffic from our VPC to the internet)
    - Type: Subnet (We are allowing resources from specific subnet i.e. private subnet to access the internet)
    - Subnet: Select *subnet-pri-red* because we want to allow only resources that sitting under private subnet to access the internet
    - EIP: Select the EIP that you just created to bind with the SNAT Rule
7. Click **OK** to create SNAT Rule

### Task 3.4 Create Security Group 

Because we will need security group specifically for each of our ECS, we need to create at least 2 security groups and attach each one of them to the ECS.

### Task 3.4.1 Create Security Group for Public ECS
1. Navigate to **Network Console**, select **Access Control** and click **Security Groups**
2. Click **Create Security Group** at the top right corner
3. On pop-up window, enter the following information:
    - Name: Enter the security group name e.g. sg-ssh-internet
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
    - Template: Custom
4. Click **OK** to create security group
5. On pop-up window, click **Manage Rule** to create security group rule
6. At the security group screen, select **Inbound Rules** and delete the default rule (you can choose to modify this rule according to the below requirement as well)
7. Click **Add Rule** and enter the following information on the pop-up windows:
    - Protocol & Port: TCP 22
    - Source: Enter your IP Address with mask. You can find from whatismyipaddress.com e.g. <Your IP Address>/32
    - Description: For Internet SSH acccess from my IP Address
8. Click **OK** to create the inbound rule

### Task 3.4.2 Create Security Group for Private ECS
1. Navigate to **Network Console**, select **Access Control** and click **Security Groups**
2. Click **Create Security Group** at the top right corner
3. On pop-up window, enter the following information:
    - Name: Enter the security group name e.g. sg-ssh-internal
    - Enterprise Project: Select the Enterprise Project that you just created (Skip this if you are using Individual Account)
    - Template: Custom
4. Click **OK** to create security group
5. On pop-up window, click **Manage Rule** to create security group rule
6. At the security group screen, select **Inbound Rules** and delete the default rule (you can choose to modify this rule according to the below requirement as well)
7. Click **Add Rule** and enter the following information on the pop-up windows:
    - Protocol & Port: TCP 22
    - Source: Select **Security Group** and choose the name of security group that you just created
    - Description: For Internal SSH acccess from sg-ssh-internet
8. Click **OK** to create the inbound rule

### Task 3.5 - Create Elastic Cloud Server (ECS)

### Task 3.5.1 - Create Public ECS

1. Navigate to **Cloud Server Console** and select **Elastic Cloud Server**
2. Click **Buy ECS** to create ECS 
3. Enter the following information for **Configure Basic Settings** screen:
    - Billing Mode: Pay-per-use
    - Region: Select the same region as your VPC
    - AZ: AZ1 
    - Specification: Leave all default option, but select **General Computing** then choose s3.small.1 (For testing purpose, we will use the smallest server size)
    - Image: Select **Public Image** and choose **Ubuntu 18.04 server 64bit (40GB)** 
    - System Disk: Leave as default
4. Click **Next: Configure Network** and enter the following information:
    - Network: Select *vpc-red* or the VPC that you have created earlier, and select *subnet-pub-red* or the public subnet that you have created earlier
    - Security Group: Select the security group that you created for public ssh access e.g. sg-ssh-internet
    - EIP: Select **Use existing** and select the EIP that you have created from previous task
5. Click **Next: Configure Advanced Settings and enter the following information:
    - ECS Name: ecs-red-bastion
    - Login Mode: **Password**
    - Password: Enter the password with the following rules:
        - Must be 8 to 26 characters long
        - Must contains at least three of the following character types:
            - Uppercase letters
            - Lowercase letters
            - Digits
            - Special characters: $!@%-_=+[]:./^,{}?
        - Cannot contain the username or the username spelled backwards.
        - Cannot contain more than two characters in the same sequence as they appear in the username. (This requirement applies only to Windows ECSs.)
    - Leave the rest as default
6. Click **Next: Confirm**, review the summary of ECS configuration. Under **Enterprise Project**, select the enterprise project that you have created earlier.
7. Check the box on **Agreement**, then click **Next**. The ECS will be created shortly. Click **Back to ECS List** and wait few minutes until the ECS becomes **running**

### Task 3.5.2 - Create Private ECS

1. Navigate to **Cloud Server Console** and select **Elastic Cloud Server**
2. Click **Buy ECS** to create ECS 
3. Enter the following information for **Configure Basic Settings** screen:
    - Billing Mode: Pay-per-use
    - Region: Select the same region as your VPC
    - AZ: AZ2
    - Specification: Leave all default option, but select **General Computing** then choose s3.small.1 (For testing purpose, we will use the smallest server size)
    - Image: Select **Public Image** and choose **Ubuntu 18.04 server 64bit (40GB)** 
    - System Disk: Leave as default
4. Click **Next: Configure Network** and enter the following information:
    - Network: Select *vpc-red* or the VPC that you have created earlier, and select *subnet-pri-red* or the public subnet that you have created earlier
    - Security Group: Select the security group that you created for private ssh access e.g. sg-ssh-internal
    - EIP: Select **Not required**
5. Click **Next: Configure Advanced Settings and enter the following information:
    - ECS Name: ecs-red-private
    - Login Mode: **Password**
    - Password: Enter the password with the following rules:
        - Must be 8 to 26 characters long
        - Must contains at least three of the following character types:
            - Uppercase letters
            - Lowercase letters
            - Digits
            - Special characters: $!@%-_=+[]:./^,{}?
        - Cannot contain the username or the username spelled backwards.
        - Cannot contain more than two characters in the same sequence as they appear in the username. (This requirement applies only to Windows ECSs.)
    - Leave the rest as default
6. Click **Next: Confirm**, review the summary of ECS configuration. Under **Enterprise Project**, select the enterprise project that you have created earlier.
7. Check the box on **Agreement**, then click **Next**. The ECS will be created shortly. Click **Back to ECS List** and wait few minutes until the ECS becomes **running**

### Task 3.6 - Configure custom route to allow egress access for the private ECS

Because the public ECS is attached to an EIP already, you can try to access the ECS via SSH with the EIP you have assigned to them without the need for routing configuration. However, the private ECS won't be able to do so, and it requires NAT Gateway to help reaching out to the internet while we are restricting the access from the internet to this private ECS. If we want to access the private ECS from the internet, we would need to leverage the public ECS that we called "Bastion Host" or "Jump Host" to perform the remote secure shell instead. 

1. Navigate to **Network Console** and select **Route Tables**
2. Click on the private route that you have created earlier e.g. *rtb-pri-red*
3. In this screen, it will show more detail about the route table. Click **Add Route** to create additional route that we need for the private ECS. 
4. On pop-up windows, enter the following information to add the route for NAT gateway:
    - Destination: 0.0.0.0/0
    - Next Hop Type: NAT Gateway
    - Next Hop: Select the NAT Gateway that you have created earlier e.g. *nat-red*
5. Click **OK** to create the route

### Task 4 - Testing the connection with ECSs

Now we have configured the route for private ECS, it should be able to reach the internet to download software package while allowing incoming remote secure shell (SSH) only from the bastion host (our public ECS). We will need to test the setup. 

### For Windows User

#### Pre-requisite
- Ensure you have Putty installed on local machine
- Ensure that you have configured the inbound rules of the security group to allow your local computer to access the ECS SSH
- Ensure that you have the network connection between the local computer to target ECS

1. Open Putty.exe and enter the following information on **Session** screen
    - Host Name (or IP Address): Enter the public bastion host IP Address that you have obtained from previous task
2. Click **Open** to start the ssh session
3. On the remote secure session screen, enter the following information:
    - Username: root
    - Password: Enter the password that you have configured for the public bastion host
4. You should be able to connect to the public bastion host via SSH successfully

### For Linux or MacOSX user

#### Pre-requisite
- Ensure that you have configured the inbound rules of the security group to allow your local computer to access the ECS SSH
- Ensure that you have the network connection between the local computer to target ECS

1. open **Terminal**
2. Type in the command below to start the ssh session
    `ssh root@<The public bastion host IP Address>`
3. Enter the password that you have configured for this ECS. You should be able to connect to the public bastion host via SSH succesfully

### Task 4.1 Testing egress connection from the private ECS

Once we have successfully connected to the public ECS, now we will remote to the private ECS and test our NAT Gateway setup with the custom route. 

1. From the SSH session, type the following command to connect to the private ECS
    ```
    ssh root@<The private ECS IP Address>
    ```
2. Enter the password that you have configure for this ECS. You should be able to connect to the private ECS successfully
3. Enter the following command to see if this private ECS can reach to the internet
    ```nslookup www.huaweicloud.com```
    `curl -I https://www.huaweicloud.com/
4. If the command return HTTP/2 200, then it means that this private ECS can reach out to the internet
5. Enter the following command to update software package in the private ECS
    `apt-get update`
    `apt-get -y upgrade`
6. Wait for few minutes until all the software package in this private ECS is completely updated

## Congratulation! Now you able to create a VPC with subnets to isolate public and private access! 