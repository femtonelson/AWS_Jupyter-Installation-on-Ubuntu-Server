[Inspired from ]

# Step 0 : Objective
- Install Jupyter on a publicly accessible Ubuntu server (AWS EC2 instance).
- Configure two different Python virtual environments with Python3.6 and Python3.7 interpreters respectively
- Each Python virtual environment runs with its own specific interpreter, libraries, scripts
- As such each virtual environment is isolated from others as well as from the default Python environment included in the operating system

# Step 1 : Launch Ubuntu Server instance on AWS inside a "public subnet", configure its Route Table, assign Public IP & Security Group
- Inside a VPC, create a suitable subnet.
- Configure its route table to allow routing to 0.0.0.0/0 through Internet Gateway.
- Create a security group to allow incoming connections on port 22 (SSH), port 8888 (port on which Jupyter server will run) and optionally ICMP (ping) port.
- Launch Ubuntu 18.04 server instance (save private key "My_key_pair.pem"), assign VPC, subnet, security group created above and public IP aaa.bbb.ccc.ddd.

# Step 2 : Connect to Server by SSH and proceed with Testing [Linux CLI used here]
In local CLI (Linux machine in this case), run :
- sudo chmod 400 My_key_pair.pem : Protect private key by making it read-only
- sudo ssh -i "My_key_pair.pem" ubuntu@aaa.bbb.ccc.ddd  :  Connect to server by SSH
- sudo apt-get update || Download the package lists for all packages on the server that need an upgrade
- sudo apt-get upgrade || Fetch on the server the new packages available and install them





The tool chosen here for Python virtual environment creation is Conda from the Anaconda repository, which natively supports env. creation with different Python versions.
Other options typically involve using "pip" (the recommended package installer) and "pew" (python environment wrapper), or pip and virtualenv. 
Read more here : https://www.anaconda.com/understanding-conda-and-pip/


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
