Creating a Virtual Private Cloud
Lab overview and objectives
Traditional networking is difficult. It involves equipment, cabling, complex configurations, and specialist skills. Amazon Virtual Private Cloud (Amazon VPC) hides the complexity, and simplifies the deployment of secure private networks.

This lab shows you how to build your own virtual private cloud (VPC), deploy resources, and create private peering connections between VPCs.

After completing this lab, you should be able to:

Deploy a VPC

Create an internet gateway and attach it to the VPC

Create a public subnet

Create private subnet

Create an application server to test the VPC

 

At the end of this lab, your architecture will look like the following example:

Architecture

 

Duration
This lab will require approximately 30 minutes to complete.

 
 

Task 1: Creating a VPC
You will begin by using Amazon VPC to create a new virtual private cloud, or VPC.

A VPC is a virtual network that is dedicated to your Amazon Web Services (AWS) account. It is logically isolated from other virtual networks in the AWS Cloud. You can launch AWS resources, such as Amazon Elastic Compute Cloud (Amazon EC2) instances, into the VPC. You can configure the VPC by modifying its IP address range, and create subnets. You can also configure route tables, network gateways, and security settings.

In the AWS Management Console, select the  Services menu, and then select VPC under Networking & Content Delivery.

The VPC console provides a wizard that can automatically create several VPC architectures. However, in this lab, you will create the VPC components manually.

In the left navigation pane, choose Your VPCs.

A default VPC is provided so that you can launch resources as soon as you start using AWS. There is also a Shared VPC that you will use later in the lab. However, you will now create your own Lab VPC.

The VPC will have a Classless Inter-Domain Routing (CIDR) range of 10.0.0.0/16, which includes all IP address that start with 10.0.x.x. It contains over 65,000 addresses. You will later divide the addresses into separate subnets.

Choose Create VPC and configure these settings:

Name tag: Lab VPC

IPv4 CIDR block: 10.0.0.0/16

Choose Create VPC

 If these options do not appear, cancel your configuration. In the left navigation pane, make sure you that chose Your VPCs. Then, choose Create VPC again.

On successful creation,following message appears at the top You successfully created vpc-NNNNNNNNN/ Lab VPC

Select Actions, then choose Edit VPC settings.

Select checkbox for Enable DNS hostnames.

This option assigns a friendly Domain Name System (DNS) name to EC2 instances in the VPC, such as:

ec2-52-42-133-255.us-west-2.compute.amazonaws.com

Choose Save.

Any EC2 instances that are launched into the VPC will now automatically receive a DNS hostname. You can also add a more meaningful DNS name (such as app.example.com) later by using Amazon Route 53.

 

Task 2: Creating subnets
A subnet is a subrange of IP addresses in the VPC. AWS resources can be launched into a specified subnet. Use a public subnet for resources that must be connected to the internet, and use a private subnet for resources that must remain isolated from the internet.

In this task, you will create a public subnet and a private subnet:

Subnets

Creating a public subnet
The public subnet will be used for internet-facing resources.

In the left navigation pane, choose Subnets.

Choose Create subnet and configure these settings:

VPC: Lab VPC

Subnet name: Public Subnet

Availability Zone: Select the first Availability Zone in the list (Do _not_choose No Preference)

IPv4 CIDR block: 10.0.0.0/24

Choose Create subnet

 The VPC has a CIDR block of 10.0.0.0/16, which includes all 10.0.x.x IP addresses. The subnet you just created has a CIDR block of 10.0.0.0/24, which includes all 10.0.0.x IP addresses. They might look similar, but the subnet is smaller than the VPC because of the /24 in the CIDR range.

You will now configure the subnet to automatically assign a public IP address for all instances that are launched in it.

Select Public Subnet.

Choose Actions and select Edit subnet settings, then:

Select  Enable auto-assign public IPv4 address under Auto-assign IP settings

Choose Save

Though this subnet is named Public Subnet, it is not yet public. A public subnet must have an internet gateway, which you attach in the next task.

 

Creating a private subnet
The private subnet will be used for resources that must remain isolated from the internet.

Use what you just learned to create another subnet with these settings:

VPC: Lab VPC

Subnet name: Private Subnet

Availability Zone: Select the first Availability Zone in the list (Do _not_choose No Preference)

IPv4 CIDR block: 10.0.2.0/23

The CIDR block of 10.0.2.0/23 includes all IP addresses that start with 10.0.2.x and 10.0.3.x. This is twice as large as the public subnet because most resources should be kept private, unless they specifically must be accessible from the internet.

Your VPC now has two subnets. However, the public subnet is totally isolated and cannot communicate with resources outside the VPC. You will next configure the public subnet to connect to the internet via an internet gateway.

 

 

Task 3: Creating an internet gateway
An internet gateway is a horizontally scaled, redundant, and highly available VPC component. It allows communication between the instances in a VPC and the internet. It imposes no availability risks or bandwidth constraints on network traffic.

An internet gateway serves two purposes:

To provide a target in route tables that connects to the internet

To perform network address translation (NAT) for instances that were assigned public IPv4 addresses

In this task, you will create an internet gateway so that internet traffic can access the public subnet.

In the left navigation pane, choose Internet Gateways.

Choose Create internet gateway and configure these settings:

Name tag: Lab IGW

Choose Create Internet gateway

You can now attach the internet gateway to your Lab VPC.

Make sure that you are on the Lab IGW window.

Choose Actions then Attach to VPC, and configure these settings:

VPC: From the list, select Lab VPC 

Choose Attach internet gateway

This action will attach the internet gateway to your Lab VPC. Though you created an internet gateway and attached it to your VPC, you must also configure the public subnet route table so it uses the internet gateway.

Task 4: Configuring route tables
A route table contains a set of rules, called routes, that are used to determine where network traffic is directed. Each subnet in a VPC must be associated with a route table because the table controls the routing for the subnet. A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same route table.

To use an internet gateway, a subnet's route table must contain a route that directs internet-bound traffic to the internet gateway. If a subnet is associated with a route table that has a route to an internet gateway, it is known as a public subnet.

In this task, you will:

Create a public route table for internet-bound traffic

Add a route to the route table to direct internet-bound traffic to the internet gateway

Associate the public subnet with the new route table

In the left navigation pane, choose Route Tables.

Several route tables are displayed, but there is only one route table associated with Lab VPC. This route table routes traffic locally, so it is called a private route table.

In the VPC ID column, select  the route table that shows Lab VPC (you can expand the column to see the names).

In the Name column, choose  then enter the name  Private Route Table and choose Save.

In the lower half of the page, choose the Routes tab.

There is only one route. It shows that all traffic that is destined for 10.0.0.0/16 (which is the range of the Lab VPC) will be routed locally. This route allows all subnets in a VPC to communicate with each other.

You will now create a new public route table to send public traffic to the internet gateway.

Choose Create route table and configure these settings:

Name: Public Route Table

VPC: Lab VPC

Choose Create route table

Ensure that you are on the Public Route Table window.

In the Routes tab, choose Edit routes

You will now add a route to direct internet-bound traffic (0.0.0.0/0) to the internet gateway.

Choose Add route then configure these settings:

Destination: 0.0.0.0/0

Target: Select Internet Gateway and then, from the list, select Lab IGW

Choose Save changes


The last step is to associate this new route table with the public subnet.

Choose the Subnet associations tab.

Choose Edit subnet associations

Select  the row with Public Subnet.

Choose Save associations

The public subnet is now public because it has a route table entry that sends traffic to the internet via the internet gateway.

To summarize, you can create a public subnet by following these steps:

Create an internet gateway

Create a route table

Add a route to the route table that directs 0.0.0.0/0 traffic to the internet gateway

Associate the route table with a subnet, which thus becomes a public subnet

 

Task 5: Creating a security group for the application server
A security group acts as a virtual firewall for instances to control inbound and outbound traffic. Security groups operate at the level of the elastic network interface for the instance. Security groups do not operate at the subnet level. Thus, each instance can have its own firewall that controls traffic. If you do not specify a particular security group at launch time, the instance is automatically assigned to the default security group for the VPC.

In this task, you will create a security group that allows users to access your application server via HTTP.

In the left navigation pane, choose Security Groups.

Choose Create security group and configure these settings:

Security group name: App-SG

Description: Allow HTTP traffic

VPC: Lab VPC

Choose Create security group

Select App-SG.

Choose the Inbound rules tab.

The settings for Inbound rules determine what traffic is permitted to reach the instance. You will configure it to permit HTTP (port 80) traffic that comes from anywhere on the internet (0.0.0.0/0).

Choose Edit inbound rules

Choose Add rule and then configure these settings:

Type:  HTTP

Source: Anywhere

Description: Allow web access

Choose Save rules

You use this App-SG in the next task.


Task 6: Launching an application server in the public subnet
To test that your VPC is correctly configured, you will now launch an EC2 instance into the public subnet. You will also confirm that you can access the EC2 instance from the internet.

In the AWS Management Console, select the  Services menu, and then select EC2 under Compute.

Choose Launch instance and select Launch instance from the drop down list.

On Launch an instance, configure following options:

In the Name and tags section, for Name, enter App Server.

Under Application and OS Images (Amazon Machine Image), In the AMI Machine Image (AMI) box, notice that Amazon Linux 2023 AMI is selected by default. Keep this setting.

For Instance type, keep the default instance type, t2.micro.

In the Key pair (login) section, from the Key pair name - required dropdown list, choose Proceed without a key pair.

In the Network settings section, choose Edit.

From the VPC - required dropdown list, choose Lab VPC.

For Subnet, Choose Public Subnet from dropdown.

For the Firewall (security groups) , choose Select existing security group.

Choose App-SG from dropdown.

Expand the Advanced details pane.

From IAM instance profile dropdown list, choose Inventory-App-Role

For User data, enter following code:

#!/bin/bash
# Install Apache Web Server and PHP
yum install -y httpd mysql
amazon-linux-extras install -y php7.2
# Download Lab files
#wget https://aws-tc-largeobjects.s3.amazonaws.com/CUR-TF-100-TULABS-153-lab-creating-vpc/inventory-app.zip
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-PRYODA-1-37918/4-lab-creating-vpc/s3/inventory-app.zip
unzip inventory-app.zip -d /var/www/html/
# Download and install the AWS SDK for PHP
wget https://github.com/aws/aws-sdk-php/releases/download/3.62.3/aws.zip
unzip aws -d /var/www/html
# Turn on web server
chkconfig httpd on
service httpd start
From the right pane, choose Launch instance.

Choose View all instances.

Wait for the application server to fully launch. It should display the following status:

Instance State:  running

 You can choose to refresh  occasionally to update the display.

Select  App Server.

Copy the Public IPv4 address  address from the Details tab.

Open a new web browser tab with that IP address using HTTP. Example http://<public-ipv4-address>/


If you configured the VPC correctly, the Inventory application and this message should appear: Please configure settings to connect to database. You have not configured any database settings yet, but the appearance of the Inventory application demonstrates that the public subnet was correctly configured.

 If the Inventory application does not appear, wait 60 seconds and refresh  the page to try again. It can take a couple of minutes for the EC2 instance to boot and run the script that installs software.