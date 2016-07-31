# README

## Introduction

This is an AWS CloudFormation template that can be used to launch a complete
[myBB](https://www.mybb.com/) stack, featuring basic security settings, logs
storage and system monitoring.

Also, it can be used as template for deploying similar stacks like: PHP + Apache
+ MySQL under segmented AWS VPCs and AWS Subnets, featuring some monitoring with
NewRelic and logs viewing/storage with AWS CloudWatch.

## Requirements

There are 2 requirements that you must fill to get the whole stack properly working:

- AWS Account: You need a AWS Account and enough privileges in order to launch a
CloudFormation stack. If there is no such privilege, your request are not going
to be processed and the whole stack are not going to be created.

- 1 AWS EC2 Key Pair: You need that a Key Pair be available to be inserted into
your brand new EC2 instances. Without that, you are not going to be able to
connect into the EC2 instances by means of SSH. So make sure you have one.

There is a monitoring requirement that is optional but highly recommended to met.
Create a New Relic account and make sure you get an access key:

- Inside New Relic dashboard, click into your avatar and name, which are located
in the up right corner from the screen, then click in "Account settings". Then
you must get the "License key" [2] that is located bellow the "Account
information" section.


## Deployment Procedures

### Shortcut to Steps 1..6

Click on the following shortcut to get the steps 1..6 accomplished:

[<img src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png">](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=my-bb-stack&templateURL=https://raw.githubusercontent.com/rpherrera/mybb-cfn/1806-mysql/mybb-cfn-stack.json)

It will take you to the launching phase from a myBB stack (version 1806), with a
MySQL database in us-east-1 AWS region.

### Full Procedure

In order to create a myBB stack, follow the instructions bellow:

  1. Log into AWS Console.

  2. Goto "Services" -> "CloudFormation".

  3. Click in "Create Stack".

  4. In "Choose template" section, mark "Upload a template to Amazon S3".

  5. Within "Upload a template to Amazon S3" section, click in "Browse".

  6. Choose the file "mybb-cfn-stack.json" located in this package.

  7. Click in "Next".

  8. Fill all the form fields, making sure you defined:
    - Stack name
    - DBPassword
    - KeyName
    - NewRelicLicenseKey (if applicable) [2]
    - OperatorEMail [3]

  9. Click "Next".

  10. In "Options" step you can leave all as is and then click in "Next"

  11. In "Review" step you are going to be required to accept the following
  capabilities "AWS::IAM::InstanceProfile" and "AWS::IAM::Role", so scroll down
  the web page, check a box right the message "I acknowledge that this template
  the cause AWS CloudFormation to create IAM resources."

  12. Click in "Create".

  13. Check your Mail Box for a message with the following subject:
  "AWS Notification - Subscription Confirmation", once you found that get into
  the message and click in "Confirm subscription". By doing this, every relevant
  step from CloudFormation will be informed to your E-Mail address [3].

  14. You will be redirected to the main CloudFormation screen and now you have
  to wait the stack creation. When the status moves from "CREATE_IN_PROGRESS" to
  "CREATE_COMPLETE", check it out the stack tab called "Output" and you will see
  the URL which was provided to your application by the means of a "Value"
  attributed to a "Key" called "WebsiteURL" and a comment besides saying
  "ELB URL for newly created myBB stack".

All done, you should see the myBB welcome screen. The next section will provide
you the credentials you need to get into as the board admin.


## myBB Default Credentials

Once the setup is done, you can sign into myBB with a default credential that is
meant to be used by the application administrator. Into the login page, just
fill the form with the following values:

 - Username: admin

 - Password: 4dm1np4ss0rd

By using the credentials above, you should get logged as the board admin. So you
can access the restrict content and do admin stuff like backups and change the
topics organization.

PLEASE CHANGE THE CREDENTIALS right after you get logged into the system.

## Design Goals

Some goals were stablished to get the solution evaluated and for every one of
them here is a briefly description explaining how we achieve that goals:

  1. VPC and subnets deployment
    One VPC is created by the CloudFormation template along 2 public subnets and
    2 private subnets, which were respectively dedicated to WebServers/ELBs and
    RDS Database machines. By segmenting the network this way, we ensure that
    RDS Database machines are not exposed to the world and the traffic must pass
    from one network to another, exclusively.

  2. Security groups configuration
    Three Security groups are intent to control the whole solution. The
    "LoadBalancerSecurityGroup" allows only TCP/80 traffic for Inbound and
    Outbound, being the door for the Internet, so the requests does not came
    directly into WebServers, which are located behind the Elastic Load
    Balancer. "WebServerSecurityGroup" allows the TCP/80 for Inbound coming
    only from the hosts that belongs to "LoadBalancerSecurityGroup", so if you
    are not our ELBs there is no Web page for you. Also, the
    "WebServerSecurityGroup" allows the SSH traffic from the address range you
    can inform into "SSHLocation" from CloudFormation options and it enables us
    to get our network very restrictive if we want to. The "DBEC2SecurityGroup"
    makes possible the communication between the WebServers (which are EC2
    instances) and the Database Servers (which are RDS instances), allowing only
    the traffic from TCP/3306. Please note that using RDS with VPC Security
    Groups is a modern approach, different from using the EC2-Classic model,
    which we avoided in order to get things more flexible and reliable.

  3. Securing DB in cloud
    The Database servers are instantiated inside 2 Private Subnets, that are
    located in different AvailabilityZones, so a MultiAZ deployment is possible
    within our CloudFormation stack, and if you do not want to it is ok, because
    it is also configurable by the MultiAZDatabase option located in the
    "Specify Details" screen from the stack creation steps (Section 3). The
    segmentation made into the VPC allows a strong logic separation between
    the machines, so no malicious traffic can flow from one subnet to another
    one. Also, there are Private Network ACLs, which restricts even more the
    allowed traffic, posing a hard design feature into the network configuration
    that is the fact the Security Groups can even specify a more permissive
    traffic, but if it is not allowed by the Private Network ACL related to the
    Private Subnet no damage can be done.

  4. Monitoring performance using AWS and other tools
    This feature resembles us the fact that when we have a fleet of servers it
    is a hard task to inspect them and find the properly debug messages that a
    malfunctioning application is writing to some log files. So there was
    created a monitoring using the CloudWatch log streams, which enables you to
    inspect key messages from the most important logs in the system located into
    every single server from the fleet. Just go to "CloudWatch", then "Logs" and
    select your Log Group based on the name you gave to the CloudFormation
    stack. There you gonna find every server with its own log file by service
    and you can inspect them by the range of time you wish. Just remember that
    they are retained for a period of 7 days and after that, all of them are
    properly rotated. You can see that there are an alarm configuration to
    watch for CPU utilization and if it is too high, another machine will be
    provisioned and the contrary effect is valid too when the resources are
    idle and there is a need for cool down the servers and save money. An
    integration was made to the third party tool called "New Relic Dashboard"
    and it provide us an insight full view from our server fleet point of view,
    relating CPU, Memory, Disk, Network in a more detailed fashion and we also
    have the advantage of seeing what is going on inside our application,
    getting informations about things like Slow DB Queries, Transactions, Load
    Time per page, User Page Loading, External Resources, API Calls and etc.

  5. Autoscaling
    By getting integrated to CloudWatch metrics on CPU usage, we can scale our
    WebServers up and down, depending on our needs. You can even setup into
    CloudFormation configurations [3] a variable called "WebServerCapacity" that
    says the initial number of servers our fleet will start operating. When
    a machine reaches more than 90% from CPU usage for 10 minutes, another one
    is launched. When there is no need for extra machines, the policy will
    cool down the fleet reducing its size down to a number that can sustain the
    operations.

  6. Using of RDS
    There are two options using RDS, which was configured to work with MySQL:
    MultiAZDatabase is meant to control if there will be a deployment with
    a main machine and a standalone one, providing a better result in terms of
    high availability. If there is no interest in having this kind of deploy,
    it is ok to work with a single machine into a single AZ, because it is
    optional. One thing that is important to note is the employed security,
    which was made using VPC Security Groups instead of the old generation ones,
    called DB Security Groups. The used approach is more modern and flexible.


## Known Limitations

There is one limitation related to the Elastic Load Balancer setup, which could
be done with 2 ELBs linked to EC2 instances. But for that we must assume one
DNS domain should be used to it. It is possible to determine in Route53 that a
DNS record is an alias to an A object and in this case we could setup a Master
Load Balancer to be aliased to its record and get another Load Balancer linked
as its fallback. Then, we let Route53 monitoring them and deciding who should be
responding at a given point time.


## Conclusion

From the technical point of view, the presented solution met all the design
goals and there is a room for a lot of improvements, such a more in depth
configuration, extending the functionality from this stack by using good
defaults but letting the user to get a more detailed configuration done. Also,
it could be done a work related to get Elastic Load Balancers high available too
by linking them to a Route53 record as aliases and specifying that one of them
should be the Main Load Balancer and the another one is its fallback.
