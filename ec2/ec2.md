# Elastic Cloud Compute notes
## What is EC2
EC2 is a service provided by amazon that replicates data center servers experience as closely as possible.
The core of this service is an instance of EC2 which is currently active virtual machine on server ( which we can look at as simply server ).
EC2 may be a virtualized server but all the details and  abstractions created around it make it look
like a real server. You can decide what OS, resources and etc will be used on this instance of ec2.

## AMI ( Amazon Machine Image )
AMI is simply a file containing protocol of serializing and instantiating EC2 servers with custom predefined configurations.
Amazon Machine Image is to running EC2 instance, just like a Docker image is to Docker container and what class is to an object instance from this class.
So you can create AMI for some concrete configuration of EC2 and than you can replicate instances of EC2 using these images.
When you press launch new instance on AWS UI, the first window that appears after this is window for choosing EC2 AMI. Most of us usually choose
Amazn Linux 2 AMI(HVM), SS Volume Type since it's free tier eligable.

There are different types of AMI-s: private, community, Amazon Quick Start AMI-s, AWS marketplace AMI-s.

## EC2 Instance types
There are different types of instances you can configure EC2 to use. At the moment there are approximately 75 unique types.
Choosing a correct type for your EC2 instance will be tottaly dependenat on the problem you are trying to solve and the domain
of the problem.
### Basic classification of instance types
- R.., X... - Optimized memory/RAM, which work well with memory intensive, database, caching and data analysis operations. Some of them have
up to 4 terabytes of DRAM.
- C.... - Optimized for CPU/Computing intensive applications.  Maybe be used for high end web servers and machine learning problems
where big CPU cycles are needed.
- M, T... - General purpose, Balanced family of machines, they stand between R,X and C families, so they have both memory and CPU somewhere in the middle. not too much,
not too many.
- I.... - I/O intensive and optimized machines, where local IO is needed, used for storages ( since local IO-s are mostly used in this case )
- G.... - Graphics processing optimized machines, that might be used in heavy machine learning workloads or rendering.

Some instance types will have huge RAM amount and HUGE CPU cycles, some less and you should choose wisely between them.
There are several instance type families in aws:
- General purpose family consists of types that have uniformly distributed cpu, ram, storage and etc. So basically every other resource
in this family's instance types are in the middle.

## Settings up EC2
For succesfully and securely using EC2 instances there are 3 key points to configure:
- AWS Region, in which AWS region will your EC2 instance be published, Usually for production instances Region closest to your customers is a 
correct answer. While choosing a region also an important part is that if you are have some sensitive information in your system to deploy and want
this system to fall under some specific location's ( country's ) jurisdiction than choosing AWS glolbal region closest to this location is the best solution.
EC2 resources can only be managed when you are "located within" that specific region that instance in deployed in. You can update/change location easily from console and UI.
Bear in mind that the costs and even functionality of services and features might vary between regions. It’s always a good idea to consult the most up-to-date official documentation
- VPC ( Virtual Private Cloud ) is a isolated networking entity where you must deploy your EC2 instance for securing and many more benefits. VPC have nearlly all the networking details
configurable. One common pattern for many development teams is to have several different VPC-s each for different development stage-s. let's say VPC1 - dev env. VPC2 - stage env, VPC3 - production env. This way all the environments are totally isolated frome each other and can't ahrm each other, since on development environment teams might be testing and changing configurations in an 
unpredictable manner.
- Tenancy, while creating EC2 instance you have the ability to specify tenancy model for your virtual machine.  Tenancy model is all about on how isolated physical machines you want your
programs to be run. Default tenancy model is that your program will be run on the same server that some other aws customer's. ( of course they'll be totally isolated, just simply it's sometimes the case that we need our instnaces to be run on totally isolated servers, not sharing physical servers with anyone ). with tenancy model AWS also gives you ability to configure it. 

## Block storages with EC2
When you create EC2 instance either from aws console or cli tool you have to configure storage device for your running virtual server. As for anything in AWS services
there are several alternatives for this mechanism too. In this context by storage we mean persistent storage, HDD or SSD where we can store files and blocks of data.
There re several alternatives to use as storage for EC2 instance:
- Instance Store - block storage hard drive/ssd ( you can choose whichever you want ) that is directl mounted on the physical machine that is used to run the instance of EC2.
This is mostly used to store some temporary data and not too miision critical or valuable data since lifecycle of this type of storage is hardly linked with lifecycle of EC2, so
if EC2 instance gets stopped or terminated this data gets cleared and we can't recover from it. Cached and stuff like that can be a good data domain to store here.
- EBS ( Elastic Block Storage ) - block storage mechanism that is a persistent/fail safe and distributed variant of local instance store. AWS admin/creator of this resource can specify
loads and loads of configurations for this type of storage. The main idea/differnece for EBS apart for instance storage is that it's lifecycle is larger that that of an EC2 instance. So even if EC2 instance gets stopped or terminated data in EBS gets stored and active. Also one important thing is that when using EBS you can unmount and remount EBS to another EC2 instance in the live and you can't do this using Instance Store, Since instance store is wired with the physical machine where EC2 is running.

You can attach as many EBS-es to one ec2 instance as you like, but one concrete EBS can be attached maximum up to 1 ec2 instance.

## Ip addresses
By default all created instances of ec2 have ip addresses, private ip and public ip, private ip address is persistent and is used to communicate between components within same VPC, public ip-s are ip-s  using which you can connect to ec2 instances from public network ( internet )  ( ofcourse if network configurations allow you to ), but 
one main point is that this IP address is not persistent, which means that IP address doesnt persist between rebooting of ec2 instance. So most likely your ip address before
and after rebooting of an instance will not be same.

One main question that might appear to the reader is what is the reason of having public and private ip addresses. and what can private ip address do what public ip can not.
here are some coll ideas from stackoverflow:
	If you use the private IP to communicate, traffic will stay within the VPC, it will not be routed out, the routing table will route it internally
If you use the public IP to communicate, traffic will go out to internet (through NAT or internet gateway) and come back to your VPC. This unnecessary roundtrip can
be avoided if you use the private IP
Since private IP is internal, it is more secure since there is no chance for third party to inspect/inject the traffic
No data transfer charge if the traffic stays internal to VPC (and same availability zone). But if data flows out of VPC, you need to pay for data xfer charge
Unless you expect your instance to accept traffic from outside, you should not launch your instances in public subnet or assign public IPs to it

### Elastic ip address
AWS also has a solution to the problem stated above and the solution is Elastic IP address. Elastic ip is public IP address mapped to your account and you can
use this allocated IP to map it to instnaces and not loose it between reboots.An Elastic IP address is a public IPv4 address, which is reachable from the internet.
 If your instance does not have a public IPv4 address, you can associate an Elastic IP address with your instance to enable communication with the internet. For example, this allows you to connect to your instance from your local computer.

## EC2 security
AWS gives account administrator several tools to guarantee the security of EC2 instances.
- Securty groups: which are same as fire walls for your ec2 instance networks in aws. Security groups control total packet flow from and to your ec2 instnace. By default security group will deny all incoming traffic
and permit all outgoing traffic. One common practive to use securtiy groups for is to allow packet for ssh protocol on port 22 only for those packets that have source ip address following/matching patterns 
of ip addresses from you office. One specific thing is that security groups are used specifically for configuring ec2 instance networks, there are also different tools to configure networks on a bigger scale like on subnets or even vpc scale.
we'll discuss it on VPC topic seperately. Security groups are instnace level.
- IAM roles: Identity access management. Using this service/mechanism root user of aws account can create other accounts and assing them specific roles, by which will be determined what resources can they access.
Using roles, you can give a limited number of entities (other resources or users) exclusive
access to resources like your EC2 instances. But you can also assign an IAM role to an EC2 instance so that processes running within it can access the external tools—like an RDS database instance—it needs to do its work.
- NAT devices: One common pattern to allow your ec2 reach the internet ( for updates, patches...etc ) but to prohibit internet from reaching your instance, you will create a new public subnet, place there NAT gateway, also you will
have your ec2 instances in a separate (private) subnet apart from NAT, this NAT will have  access to internet and  will prohibit packets from internet flowing into you ec2 instnaces but will allow ec2 instances to use internet. Solutions with
NAT are subnet level solutions and not single instnace level ones, so this is broader that secuirty groups. Also there might be both usued: security groups and NAT-s for double security reasons.
