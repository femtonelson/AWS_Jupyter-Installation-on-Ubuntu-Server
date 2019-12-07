# AWS Tutorials
## Setup a Jumpbox (for management) + NAT instance + private machine in AWS VPC
[Inspired from https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html]

# Step 0 : Objective
- Setup a simple AWS VPC with 2 instances in a public subnet (reachable from internet) and an EC2 instance in a private subnet, not directly accessible to the internet, its requests will be forwarded through the NAT instance.
- An EC2 instance in the public subnet called "Jumpbox", with a public IP and will serve for managing remotely all instances in the VPC
- The other instance in the public subnet called "NAT instance", with a public IP, will perform NAT translations.
- At the end of this exercise, FZ machine should be able to access the internet through the NAT instance and ping www.amazon.com or download a software package.

# Step 1 : Create a New VPC, Internet Gateway and 2 subnets 
- Create a new VPC (Virtual Private Cloud), named "New_VPC" in the IPV4 address range, for instance : 10.0.0.0/16.
- Create a new Internet Gateway "IGW" and attach to New_VPC.
- Create a public subnet inside New_VPC, named "Public subnet" in the IPv4 address rang, for instance : 10.0.4.0/24.
- Create a private subnet inside New_VPC, named "Private subnet" in the IPv4 address range, for instance : 10.0.6.0/23.

# Step 2 : Create 03 new Security Groups attached to New_VPC
- Create a new SG "Private SG": In Inbound rules set [All traffic -> All protocols -> All ports -> 10.0.0.0/16] to enable all incoming traffic from within the VPC.
  In outbound rules set [All traffic -> All protocols -> All ports -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.
- Create a new SG "Jumpbox SG": In Inbound rules set [SSH -> TCP -> 22 -> 0.0.0.0/0], [All ICMP - IPv4 -> All -> N/A -> 0.0.0.0/0] to enable incoming SSH/Pings from internet/VPC.
  In outbound rules set [All traffic -> All protocols -> All ports -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.
- Create a new SG "NAT SG": In Inbound rules set [All traffic -> All protocols -> All ports -> 10.0.0.0/16] to enable all incoming traffic from within the VPC. Add a second inbound rule [All ICMP - IPv4 -> All -> N/A -> 0.0.0.0/0] to respond to ping requests from the internet.
  In outbound rules set [All traffic -> All -> All -> 0.0.0.0/0] to enable all outgoing traffic to all destinations.

# Step 3 : Launch 03 new instances or machines [Use the same key pair, download "My_key_pair.pem" and keep it for ssh login]
- Launch a new EC2 instance to be named "Jumpbox" based on Linux 2 AMI -> Instance Type (t2.micro) -> Associate it to New_VPC and Public subnet.
- Launch a new EC2 instance to be named "FZ Machine" based on Linux 2 AMI -> Instance Type (t2.micro) -> Associate it to New_VPC and Private subnet.
- Launch a new instance to be named "NAT Instance" with suitable NAT AMI (Amazon Machine Image) from Community AMIs "amzn-ami-vpc-nat" -> Instance Type (t2.micro) -> Associate it to New_VPC and Public subnet.
- For this particular exercise, auto-assigned IPs of the 03 machines were as follows :
  Jumpbox (10.0.4.12), NAT Instance (10.0.4.124), FZ Machine (10.0.6.145)
  
# Step 4 : Create 02 Route Tables : A public route table and a private route table attached to New_VPC
- Create Route Table named "Public RT". Under Public RT -> Routes, ensure there are 2 entries : [Destination - 10.0.0.0/16   ->   Target - Local] for routing within VPC and [Destination - 0.0.0.0/0   ->   Target - IGW] for routing connections outside VPC, to the internet and under Public RT -> Subnet associations, associate to previously created Public subnet.
- Create Route Table named "Private RT". Under Private RT -> Routes, ensure there are 2 entries : [Destination - 10.0.0.0/16   ->   Target - Local] for routing within VPC and [Destination - 0.0.0.0/0   ->   Target - NAT Instance] for routing connections to the internet, through the NAT instance.

# Step 5 : Allocate 02 Elastic IP addresses, to Jumpbox and NAT instance
- Elastic IPs -->Allocate new address (IP1) --> Associate with Jumpbox
- Elastic IPs -->Allocate new address (IP2) --> Associate with NAT Instance

# Step 6 : Disable SRC/Dest Check on NAT Instance
- NAT Instance -> Networking -> Change SRC/Dest. check -> Disabled. This enables the NAT instance to send and receive traffic without being the source or destination of such traffic.

# Step 7 : Connect to Jumpbox by SSH and proceed with Testing [Linux CLI used here]
In local CLI, run :
- sudo chmod 400 My_key_pair.pem : Protect this private key by making it read-only
<img src="./1.jpg">
- sudo scp -i "My_key_pair.pem" My_key_pair.pem ec2-user@IP1:/home/ec2-user/ : Copy the private key by ssh from working dir on local 
machine to Jumpbox at IP1 into the folder /home/ec2-user, same private key is used here for ssh connection
<img src="./2.jpg">
- sudo ssh -i "My_key_pair.pem" ec2-user@IP1 : Connect by SSH to the Jumpbox -> Successful !
<img src="./Connected to JB.jpg">
Once connected to the Jumpbox, (private key already in the working directory) connect to FZ machine by running :                          - sudo ssh -i "My_key_pair.pem" ec2-user@10.0.6.145 : Connect to FZ machine  --> Successful !
<img src="./Connected to FZ.jpg">
- ping www.amazon.com  : Ping is successful! as traffic from FZ machine is routed through NAT instance, to the internet.
<img src="./Ping to AMZ successful.jpg">
- traceroute www.amazon.com : Traceroute shows the first hop is the NAT instance (10.0.4.124) --> Configuration successful !
<img src="./Traceroute.jpg">
- sudo wget https://download2.rstudio.org/rstudio-server-rhel-1.1.383-x86_64.rpm : Download a file from internet on FZ machine to check internet connectivity --> Download successful !
<img src="./Test Download.jpg">

# Step 8 : Clean-up after testing
- Release all allocated IP addresses to Amazon pool if no longer in use
- Stop or Terminate EC2 instances no longer in use
