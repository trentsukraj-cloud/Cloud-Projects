Creating an Amazon RDS Database
Lab overview and objectives
Traditionally, creating a database can be a complex process that requires either a database administrator or a systems administrator. In the cloud, you can simplify this process by using Amazon Relational Database Service (Amazon RDS).

After completing this lab, you should be able to:

Launch a database using Amazon RDS  

Configure a web application to connect to the database instance

At the end of this lab, your architecture will look like the following example:

Architecture

Duration
This lab will require approximately 20 minutes to complete.


Task 1: Creating an Amazon RDS database
In this task, you will create a MySQL database in your virtual private cloud (VPC). MySQL is a popular open source relational database management system (RDBMS), so there are no software licensing fees.

 Windows Users: Use Chrome or Firefox as your web browser for this lab. The lab instructions are not compatible with Internet Explorer because of a difference in the Amazon RDS console.

In the AWS Management Console, select the  Services menu, and then select RDS under Database.

In the left navigation pane, click Databases.

Choose Create database

Under Engine options, select  MySQL.

In the Templates section, select  Dev/Test.

You can now select a database configuration, including software version, instance class, storage, and login settings. The Multi-AZ deployment option automatically creates a replica of the database in a second Availability Zone for high availability. 

In Availability and durability section, choose  Single DB instance.

In the Settings section, configure the following options:

DB instance identifier: inventory-db

Master username: admin

Master password: lab-password

Confirm password: lab-password


In the DB instance class section, configure the following options:

Select  Burstable classes (includes t classes).

Select db.t3.micro.

In the Storage section, configure the following options:

Storage type: Select General Purpose SSD (gp2)

Allocated storage: Enter 20 GiB

In the Connectivity section, configure the following option: 

Virtual private cloud (VPC): Lab VPC

In the VPC security group (firewall) section, for Existing VPC security groups, choose the X on default to remove this security group. Then choose the dropdown list, and select DB-SG to add it.

In the Monitoring section, clear (deselect) the Enable Enhanced monitoring option.

In the Additional configuration section, choose  to expand it. 

For Initial database name, enter inventory

At the bottom of the screen, choose Create database.

Your database will now be launched.

Note: Close the option for Suggested add-ons if prompted.

Choose the link displayed the top inventory-db .

You will now need to wait approximately 4 minutes for the the database to be available. The deployment process is deploying a database in two different Availability zones.

Wait until Status changes to Modifying or Available.

Scroll down to the Connectivity & Security section and copy the Endpoint field.

It will look similar to: inventory-db.cggq8lhnxvnv.us-west-2.rds.amazonaws.com

Paste the Endpoint value into a text editor. *You will use it later in the lab*.


 

Task 2: Configuring web application communication with a database instance
This lab automatically deployed an Amazon Elastic Compute Cloud (Amazon EC2) instance with a running web application. You must use the IP address of the instance to connect to the application.

In the AWS Management Console, select the  Services menu, and then select EC2 under Compute.

In the left navigation pane, choose Instances.

In the center pane, there should be a running instance that is named App Server.

Select the App Server instance.

In the Description tab, copy the IPv4 Public IP to your clipboard.

Tip: If you hover over the IP address, a copy  icon appears. To copy the displayed value, choose the icon.

Open a new web browser tab, paste the IP address into the address bar, and then press ENTER.

The web application should appear. It does not display much information because the application is not yet connected to the database.

Choose Settings.

You can now configure the application to use the RDS DB instance you created earlier as follows:.

Endpoint: Paste the endpoint you copied earlier

Database: inventory

Username: admin

Password: lab-password

Choose Save


The application will now connect to the database, load some initial data, and display information.

  Add inventory,  edit, and delete inventory information by using the web application.

The inventory information is stored in the Amazon RDS MySQL database that you created earlier in the lab. This means that any failure in the application server will not lose any data. It also means that multiple application servers can access the same data.

Insert new records into the table. Ensure that the table has 5 or more inventory records before submitting your work.

 You have now successfully launched the application and connected it to the database!


Optional: You can access the saved parameters in the Systems Manager console, under Parameter Store.