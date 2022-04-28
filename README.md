# AWS LAB 52

- [AWS LAB 52](#aws-lab-52)
  - [Build Your VPC and Launch a Web Server](#build-your-vpc-and-launch-a-web-server)
  - [Scenario](#scenario)
  - [Task 1: Create Your VPC](#task-1-create-your-vpc)
    - [In the AWS Management Console](#in-the-aws-management-console)
  - [Task 2: Create Additional Subnets](#task-2-create-additional-subnets)
    - [Create A Second `Public` Subnets](#create-a-second-public-subnets)
    - [Create A Second `Private` Subnet.](#create-a-second-private-subnet)
    - [Route Table](#route-table)
  - [Task 3: Create a VPC Security Group](#task-3-create-a-vpc-security-group)
  - [Task 4: Launch a Web Server Instance](#task-4-launch-a-web-server-instance)
    - [Name and Tags](#name-and-tags)
    - [Application and OS Images](#application-and-os-images)
    - [Instance Type](#instance-type)
    - [Key Pair](#key-pair)
    - [Network](#network)
    - [Firewall Security](#firewall-security)
  - [Configure Storage](#configure-storage)
    - [Advanced Details](#advanced-details)
    - [Connect to the web server running on the EC2 instance.](#connect-to-the-web-server-running-on-the-ec2-instance)
    - [Lab Complete](#lab-complete)

## Build Your VPC and Launch a Web Server

In this lab, you will use Amazon Virtual Private Cloud (VPC) to create your own VPC and add additional components to produce a customized network. You will also create security groups for your EC2 instance. You will then configure and customize an EC2 instance to run a web server and launch it into the VPC.

Amazon Virtual Private Cloud (Amazon VPC) enables you to launch Amazon Web Services (AWS) resources into a virtual network that you defined. This virtual network closely resembles a traditional network that you would operate in your own data center, with the benefits of using the scalable infrastructure of AWS. You can create a VPC that spans multiple Availability Zones.

## Scenario

In this lab you build the following infrastructure:

- Architecture
- Objectives

After completing this lab, you can:

- Create a VPC.
- Create subnets.
- Configure a security group.
- Launch an EC2 instance into a VPC.
- Duration

This lab takes approximately 45 minutes to complete.
 
## Task 1: Create Your VPC

In this task, you will use the VPC Wizard to create a VPC an Internet Gateway and two subnets in a single Availability Zone. An Internet gateway (IGW) is a VPC component that allows communication between instances in your VPC and the Internet.

After creating a VPC, you can add subnets. Each subnet resides entirely within one Availability Zone and cannot span zones. If a subnet's traffic is routed to an Internet Gateway, the subnet is known as a public subnet. If a subnet does not have a route to the Internet gateway, the subnet is known as a private subnet.

The wizard will also create a NAT Gateway, which is used to provide internet connectivity to EC2 instances in the private subnets.

### In the AWS Management Console

- Select `Services` Menu
- Select `Networking & Content Delivery`
- Select `VPC`
- Click Launch VPC Wizard
- In the left navigation pane, click `VPC with Public and Private Subnets` (the second option).
- Click Select then configure:

```
VPC name.................: Lab VPC
Availability Zone........: Select the first Availability Zone
Public subnet name.......: Public Subnet 1
Availability Zone........: Select the first Availability Zone (the same as used above)
Private subnet name......: Private Subnet 1
Elastic IP Allocation ID.: Click in the box and select the displayed IP address
```
- Click Create VPC
- The wizard will create your VPC. Once it is complete, click OK

**Note:** The wizard has provisioned a VPC with a `Public Subnet` and a `Private Subnet` in the same `Availability Zone`, together with `Route Tables` for each subnet.

**Note:** The **Public Subnet** has a `CIDR` of `10.0.0.0/24`, which means that it contains all IP addresses starting with `10.0.0.x.`

**Note:** The **Private Subnet** has a `CIDR` of `10.0.1.0/24`, which means that it contains all IP addresses starting with `10.0.1.x`.

## Task 2: Create Additional Subnets

In this task, you will create two additional subnets in a second Availability Zone. This is useful for creating resources in multiple Availability Zones to provide High Availability.
### Create A Second `Public` Subnets

- In the left navigation pane, click `Subnets`.
- Click `Create Subnet` then configure with the following:

```
VPC ID............: Lab VPC
Subnet name.......: Public Subnet 2
Availability Zone.: Select the second Availability Zone
IPv4 CIDR block...: 10.0.2.0/24
```

**Note:** The subnet will have all IP addresses starting with 10.0.2.x.

- Click `Create Subnet`.

### Create A Second `Private` Subnet.

- Click `Create Subnet` then configure with the following:

```
VPC ID............: Lab VPC
Subnet name.......: Private Subnet 2
Availability Zone.: Select the second Availability Zone
CIDR block........: 10.0.3.0/24
```
**Note:** The subnet will have all IP addresses starting with 10.0.3.x.

- Click `Create Subnet`.

### Route Table

You will now configure the Private Subnets to route internet-bound traffic to the NAT Gateway so that resources in the Private Subnet are able to connect to the Internet, while still keeping the resources private. This is done by configuring a Route Table.

A route table contains a set of rules, called routes, that are used to determine where network traffic is directed. Each subnet in a VPC must be associated with a route table; the route table controls routing for the subnet.

- In the left navigation pane, click `Route Tables`.
- Select the route table with `Main = Yes` and `VPC = Lab VPC`. 
  
**Note:** (Expand the VPC ID column if necessary to view the VPC name.)

- In the lower pane, click the `Routes` tab.

**Note:** `Destination 0.0.0.0/0` is set to `Target nat-xxxxxxxx`. This means that traffic destined for the internet `(0.0.0.0/0)` will be sent to the `NAT Gateway`. The `NAT Gateway` will then forward the traffic to the internet.

This `Route Table` is therefore being used to route traffic from `Private Subnets`. You will now add a name to the `Route Table` to make this easier to recognize in future.

- In the `Name` column for this route table, click the pencil then type `Private Route Table` 
- Click in the lower pane and click the `Subnet Associations` tab.

You will now associate this route table to the `Private Subnets`.

- Click `Edit Subnet Associations`.
- Select both `Private Subnet 1` and `Private Subnet 2`.

**Note:** You can expand the Subnet ID column to view the Subnet names.

- Click `Save Associations`.

You will now configure the Route Table that is used by the Public Subnets.

- Select the `Route Table` with `Main = No` and `VPC = Lab VPC` (and deselect any other subnets).
- In the `Name` column for this route table, click the pencil then type `Public Route Table`.
- Click in the lower pane and click the `Routes` tab.

 **Note:** that `Destination 0.0.0.0/0` is set to `Target igw-xxxxxxxx`, which is the `Internet Gateway`. This means that internet-bound traffic will be sent straight to the internet via the `Internet Gateway`.

 You will now associate this route table to the `Public Subnets`.

- Click the `Subnet Associations` tab.
- Click `Edit Subnet Associations`
- Select both `Public Subnet 1` and `Public Subnet 2`.
- Click `Save Associations`

 **Note:** Your VPC now has public and private subnets configured in two Availability Zones:

## Task 3: Create a VPC Security Group

In this task, you will create a VPC security group, which acts as a virtual firewall. When you launch an instance, you associate one or more security groups with the instance. You can add rules to each security group that allow traffic to or from its associated instances.

- In the left navigation pane, click `Security Groups`.
- Click `Create Security Group` and then configure with the following:

```
Security Group Name.: Web Security Group
Description.........: Enable HTTP access
VPC.................: Lab VPC
```

**Note:** You will now add a rule to the security group to permit inbound web requests.

- In the Inbound rules section, click `Add Rule`, then configure with the following:

```
Type........: HTTP
Source......: Anywhere-IPv4
Description.: Permit web requests
```

- Scroll to the bottom of the screen and click `Create Security Group`

**Note:** You will use this security group in the next task when launching an Amazon EC2 instance.
 

## Task 4: Launch a Web Server Instance

In this task, you will launch an Amazon EC2 instance into the new VPC. You will configure the instance to act as a web server.

- In the `AWS Management Console`, select the `Services` menu, `Compute` and `EC2`.
- Click `Launch Instance`.

### Name and Tags

- Create a tag with the following:

```
Key........: Name
Value......: Web Server 1
```

### Application and OS Images

**Note:**  Leave everything with default settings

- Select an `Amazon Machine Image (AMI)`. *Use default*
- In the row for `Amazon Linux 2` (at the top), click `Select`. *Use default*

### Instance Type

**Note:** Leave everything with default settings

- Select `t2.micro`. *Use default*

### Key Pair

- Select `vockey`
### Network

- For `VPC`, select `vpc-xxxxxxxxxxxxx (Lab 1)`
- For `Subnet` select `Public Subnet 2`
- For `Auto assign public IP` select `Enable`

### Firewall Security

- Select an existing security group
- Select `Web Security Group`.

```
Network...............: Lab VPC
Subnet................: Public Subnet 2 (not Private!)
Auto-assign Public IP.: Enable
```

**Note:** This is the security group you created in the previous task. It will permit HTTP access to the instance.

## Configure Storage

**Note:** Leave everything with default settings

- 1 x 8 GiB
- Root Volume: `gp2`

### Advanced Details

- Expand the `Advanced Details` section (at the bottom of the page).
- Copy and paste the following code into the `User Data` box:

```bash
#!/bin/bash
# Install Apache Web Server and PHP
yum install -y httpd mysql php
# Download Lab files
#wget https://aws-tc-largeobjects.s3.amazonaws.com/ILT-TF-100-TUFOUN-1/4-lab-vpc-web-server/lab-app.zip
wget %% S3_HTTP_PATH_PREFIX %%/lab-app.zip
unzip lab-app.zip -d /var/www/html/
# Turn on web server
chkconfig httpd on
service httpd start
This script will be run automatically when the instance launches for the first time. The script loads and configures a PHP web application.
```

- On the right, click `Launch Instance`
- Click on `View all instances`

- When prompted with a warning that you will not be able to connect to the instance through port 22, click Continue

- Review the instance information and click Launch
- In the Select an existing keypair dialog, select `I acknowledge`....
- Click `Launch Instances` and then click `View Instances`
- Wait until Web Server 1 shows `2/2 checks passed` in the `Status Checks` column.
- This may take a few minutes. Click `refresh` in the top-right every 30 seconds for updates.

 ### Connect to the web server running on the EC2 instance.

- Copy the `Public (IPv4) DNS` value shown in the `Details` tab at the bottom of the page.
- Open a new web browser tab, paste the `Public DNS` value and press `Enter`.
- You should see a web page displaying the AWS logo and instance meta-data values. 
  
 **Note:** If the webpage does not load, then ensure `Status checks have passed` and also ensure you are using `http://` instead of `https://` in the web browser.
 
### Lab Complete 

Congratulations! You have completed the lab.