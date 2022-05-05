# EC2 Load Balancers

## Template base Application Load Balancer

Steps:

1. Create a `Lauch Template`
1. Create an `Auto Scaling Groups` and associate the previous `Launch Template`
1. Create a `Target Group` and assign it inside the previous `Auto Scaling Group`
1. Create a `Application Load Balancer` and assign it to the `Target Group` or inside the `Auto Scaling Group`

### 1. Lauch Template

The `Lauch Template` describes the instances that will be used in the autoscaling group.

- Select an AMI (Amazon Machine Image) - e.g. an Amazon Linux 2
- Select an instance type - e.g. t2.micro
- Assign a security group - e.g. allow HTTP/HTTPS/SSH
- Storage - e.g. 8GB gp2 as Root volumne
- Add `User data` in `Advance details` section

As example, each instance is an NGINX web server with an `index.html` page that shows some instance meta data. Include this in the `User data` section.

```bash
#!/bin/bash
yum update -y
sudo amazon-linux-extras install nginx1 -y
systemctl start nginx
systemctl enable nginx
EC2ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
echo "<h1>Amazon EC2 instance in $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)</h1>" >  /usr/share/nginx/html/index.html
echo "<ul><li>The AWS Region: $(curl -s http://169.254.169.254/latest/meta-data/placement/region) </li>" >>  /usr/share/nginx/html/index.html
echo "<li>Availability Zone (and its ID): $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)" >>  /usr/share/nginx/html/index.html
echo "($(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone-id)) </li>" >>  /usr/share/nginx/html/index.html
echo "<li>The Instance ID is: $(curl -s http://169.254.169.254/latest/meta-data/instance-id) </li>" >>  /usr/share/nginx/html/index.html
echo "<li>The AMI ID used to launch the instance: $(curl -s http://169.254.169.254/latest/meta-data/ami-id) </li>" >>  /usr/share/nginx/html/index.html
echo "<li>The type of instance: $(curl -s http://169.254.169.254/latest/meta-data/instance-type) </li><ul>" >>  /usr/share/nginx/html/index.html
echo "<p>hostanme $(hostname -f)</p>" >>  /usr/share/nginx/html/index.html
```

### 2. Create an Auto Scaling group from the template

Amazon EC2 Auto Scaling helps to maintain application availability and allows to scale the EC2 capacity up or down automatically according to some conditions.

- Use previous defined `Launch template`
- Define in `Network` the VPC and one or more Availability Zones
- Optionally here we could attach a `Load Balancer`
- Configure group size and scaling policies
  - desired, minimum and max capacity
  - Scaling policy based on CPU usage

### 2. (optional) Create Spot Fleet

Instead of creating an Auto Scaling Group, we could create a Spot Fleet

A Spot Instance is an unused EC2 instance that is available for less than the On-Demand price.

- Use previous defined `Launch template`
- Define `Target capacity`.
  - The target capacity can be defined with number of instances or vCPUs
  - You can allocate part of target capacity as On-Demand instance.
  - You can also automatically replace interrupted spot instances to maintain the capacity
- Define `Network` the specific Availability Zones or balance across all AZ

### 3. Create a Target Group

A target group tells a load balancer where to direct traffic to : EC2 instances, fixed IP addresses; or AWS Lambda functions, amongst others.

- Choose target type - e.g. instances to use the EC2 Auto Scaling
- Define `Protocol` - e.g. HTTP / HTTPS (and protocol version HTTP/1.1, HTTP/2 or gRPC)
- Associate `Health checks` to test the target status
- DO NOT ASSIGN TO ANY INSTANCE

Go to the `Auto Scaling group` and assign the new `Target group` under Load Balancing option.

### 4. Create the Load Balancer

We can choose between the following Load Balancers:

- Application (L7) - provide advanced routing
- Network (L4) - ultra-high performance, TLS
- Gateway (L7) - improve security

- Select internet-facing or internal LB
- In the newtwork mapping select the AZ - e.g. at least two AZ
- Assign a `Security Group` - e.g. allow HTTP traffic
- Listeners rules direct the traffic to one target group
