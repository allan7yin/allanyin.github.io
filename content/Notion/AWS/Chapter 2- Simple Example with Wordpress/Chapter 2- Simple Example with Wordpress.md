We’ll look at a simple example of migrating a web application to AWS by setting up sample cloud infrastructure. We’ll use 5 main core AWS services to copy the old infrastructure to AWS:

- **Elastic Load Balancing (ELB)**
    - AWS offers a load balancer as a service → distributes traffic to virtual machines and highly available by default. Requests are routed to virtual machines as long as their health check succeeds.
- **Elastic Compute Cloud (EC2)**
    - The EC2 is a web service provides virtual machines known as “instances”. In this example, we’ll use a Linux machine to install Apache, PHP, and WordPress. Virtual machines can fail, and so, we need at least 2 of them.
    - The load balancer will distribute the traffic between them → if one VM fails, balancer will stop directing traffic to failed VM, and remaining VM will handle all requests until failed VM is replaced
- **Relational Database Service (RDS) for MySQL**
    - WordPress relies on MySQL, and AWS provides MySQL with its RDS.
- **Elastic File System**
    - WordPress itself consists of PHP and other application files (e.g images and other user uploads). With this, VM’s can access these files. EFS provides scalable, highly available, and durable network filesystems
- **Security Groups**
    - This is a firewall that controls incoming and outgoing traffic to VM’s database, or load balancer. For example, use a security group allowing incoming HTTP traffic from the internet to port 80 of the load balancer. Or restrict network access to your database on port 3306 to the virtual machines running your web servers.

The overall infrastructure will look something like:

![[Screenshot_2023-12-25_at_3.41.50_PM.png]]

  

We can create all of this very quickly using **AWS CloudFormation service.** This is a service that allows us to define and provision AWS infrastructure as code. Instead of manually creating and configuring resources in the AWS Management console, we can use **CloudFormation** templates to describe the architecture and resources you need. To view the steps to accomplish this, refer to the textbook chapter 2. To reiterate, AWS CloudFormation service does all the following in the background:

- Creates a load balancer (ELB)
- Created a MySQL database (RDS)
- Creates a network file system (EFS)
- Creates and attaches firewall rules (security groups)
- Creates 2 virtual machines:
    - creates 2 virtual machines (EC2)
    - mounts the network file system (EFS)
    - Installs Apache and PHP
    - Downloads and extracts the 4.8 release of WordPress
    - Configures WordPress to use the created MySQL database (RDS)
    - Starts up the Apache web server

![[Screenshot_2023-12-25_at_7.10.35_PM.png]]

AWS CloudFormation templates are a form of Infrastructure as Code (IaC). Infrastructure as Code is an approach to managing and provisioning computing infrastructure through machine-readable script files rather than through physical hardware configuration or interactive configuration tools. **We’ll see more of this later on.** A "stack" typically refers to a collection of AWS resources that you can manage as a single unit.

---

### Exploring infrastructure

We have finished created our blogging infrastructure. Let’s take a closer look at it.

  

**→ Virtual Machines**

In AWS, each virtual machine is called an **“EC2 instance”**. In this working example, we created 2 virtual machines. For each instance, some interesting details are:

- **InstanceID →** ID of the virtual machine
- **Instance Type →** size of the virtual machine (CPU + Memory)
- **IPv4 Public IP →** IP address that is reachable over the internet
    - IPv4 is just — **Internet Protocol Version 4** — the IP address of the VM
- **AMI ID →** Remember that we are using Amazon linux OS

![[Screenshot_2023-12-25_at_7.21.23_PM.png]]

![[Screenshot_2023-12-25_at_7.36.41_PM.png]]

  

**→ Load Balancer**

As mentioned before, the load balancer forwards an incoming request to one of the virtual machines. A target group is used to define the targets of a load balancer. What is a target group? **Grouping targets** are a collection of targets, which can be virtual machines, IP addresses, or other instances. A target group is a logical grouping of targets that the load balancer routes traffic to. There is a side bar, as seen above, that shows the target groups. Above is what is seen in the dashboard.

  

**→ MySQL Database**

To view this, we open the RDS via the main navigation.

![[Screenshot_2023-12-25_at_7.48.52_PM.png]]

As seen in the above, the RDS is using a SSD storage and we have 5GB available → more than enough for a simple word press application

We’ll see later on, that other database engines such as PostgreSQL are also available. Wordpress also stores data outside the database on disk. For example, if an author uploads an image for their blog post, the file is stored on disk.

  

**→ Network File System**

This is used to store files and access them from multiple virtual machines. In simple terms, all files that belong to WordPress, including HTML, PHP, CSS, and PNG files, are stored on the EFS so that they can be accessed by any virtual machine.

- Amazon EFS offers two throughput modes: "Standard" and "Bursting” → won’t go into detail what the differences between these are

To mount the Elastic File System from a virtual machine, mount targets are needed. You should use two mount targets for fault tolerance. The network filesystem is accessible using a DNS name for the virtual machines.

  

**→ Deleting infrastructure**

So we’ve seen the quick example of migrating the company’s blogs to AWS from a technical standpoint. To delete this stack, simply navigate back to the page for the stack and delete.

---

### Summary

- Creating a cloud infrastructure for WordPress and any other application can be fully automated.
- AWS CloudFormation is a tool provided by AWS for free. It allows you to automate the managing of your cloud infrastructure.
- The infrastructure for a web application like WordPress can be created at any time on demand, without any up-front commitment for how long you’ll use it.
- You pay for your infrastructure based on usage. For example, you pay for a virtual machine per second of usage.
- The infrastructure required to run WordPress consists of several parts, such as virtual machines, load balancers, databases, and network filesystems.
- The whole infrastructure can be deleted with one click. The process is powered by automation.