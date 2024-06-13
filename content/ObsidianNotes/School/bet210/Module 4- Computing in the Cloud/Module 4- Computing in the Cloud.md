In the previous module, we explored a simple network and some of the underlying concepts. In this module, we will continue to explore important concepts related to servers and what is known as the cloud.

  

### Key Terms

- **Cloud:** On-demand delivery of computing resources through the internet.
- **Software Environment:** This is all the software required to support an application, such as the operating system, associated databases, drivers, etc.
- **Virtualization:** Virtualization is the creation of a virtual computer system that resides within another physical device.
- **Virtual Machine:** A virtual machine is a computing system that resides within another physical device. This allows multiple computing systems to reside on a single physical device.

  

## 4b. Acquiring or Supporting Servers

An issue that emerged during the launch of the Phoenix system was the number of servers required to move the project into production. This results in a cost to the organization for acquiring the server computing hardware and software, as well as the creation of physical space to house them and the support staff. In the story,The Phoenix Project,. several important considerations were discussed when acquiring and supporting servers for an enterprise, as shown in the table below.

- **Scalability:** "Scalability is an attribute that describes the ability of a process, network, software or organization to grow and manage increased demand. A system, business or software that is described as scalable has an advantage because it is more adaptable to the changing needs or demands of its users or clients" ("What is Scalability?", n.d.).
- **Availability:** "Availability, in the context of a computer system, refers to the ability of a user to access information or resources in a specified location and in the correct format" ("What is Availability?", n.d.).
- **Survivability:** The survivability of a system is its ability to remain functional in the presence of a threat, such as a cyberattack, or other major disruption.
- **Security:** A broad term covering the protection and authentication of information in a system.
- **Supportability:** The ability of an organization to operate and maintain a system over its expected lifetime.

  

## 4c. The Cloud

**⇒ Cloud Computing**

A solution to the infrastructure costs and these other considerations is the use of cloud computing. In a previous module, we considered a server physically residing in an enterprise’s facilities. In this case, the firm incurs the cost associated with acquiring, installing and supporting the associated hardware and software. This can become very expensive, especially for small companies or start-ups.

An option to dealing with these costs and many of the considerations discussed previously is to move the firm’s infrastructure to the cloud. Recall the diagram where we developed a simple network. In that example, the journey ended at the service provider’s server, which was physically located at their location. This is true whether that is Toronto, London, Delhi, or Tokyo.

The figure **Simple Cloud Implementation** illustrates the implementation of the cloud. In the cloud, physical servers are not located at the end service provider. Rather they are housed at a third party location; this may be local or in another geographic location. In this case, the end service provider can use the computing resources as needed and manage them remotely without being concerned with the operational aspects, such as maintenance and facilities, for these resources.

![[Screenshot_2023-02-02_at_5.41.46_PM.png]]

**⇒ Cloud Service Providers and Configurations**

There are several large providers of cloud services, such as Amazon Web Service (AWS), Microsoft Azure, Google, and IBM. For a firm, cloud computing provides on-demand computing services on a pay-as-you-go basis. This is often referred to as “server-less” computing since the enterprise only needs to provide the software without considering the server itself. Further, there are different types of cloud services available, shown in the following table, which can be ‘rented’ depending on the firm's requirements.

![[Screenshot_2023-02-02_at_5.44.29_PM.png]]

Cloud configurations, such as AWS, can be referred to as a public cloud. This implies that anyone can have access to the infrastructure. When reviewing the key considerations of acquiring or supporting a server, it may be that the requirements for security, availability, etc, may not be fully met by a public cloud.

This may lead to the implementation of a private cloud system. A private cloud system is exclusive to the user and operated for a single user. This provides the benefits of a public cloud configuration, but without relinquishing control over issues such as data security.

Service providers can often host Virtual Private Networks (VPN) that allow the user to define the network and control access. This has the benefits of a private network, as would be implemented in the firm’s facilities, as well as the availability and scalability of cloud computing. A third configuration is a hybrid cloud, which has characteristics of both a private and public cloud. An example of this type of system could be one in which processes with high operational importance are placed on a private cloud system, say factory operations and finance, while other functions such as customer relationship management are accessed through a SaaS service like Salesforce.

  

## 4d. Virtualization and Virtual Machines

An issue that comes up in The Phoenix Project story is virtualization. This was seen to be a solution to many of the hardware system issues being faced by the organization. The basic idea behind the virtualization is to allow a single physical server to execute many different, apparently independent applications simultaneously. To achieve this, virtual machines (VM) are created on the physical server.

![[Screenshot_2023-02-02_at_5.50.18_PM.png]]

We have examined the software side of a server, as shown in section A of the figure. In the case of virtualized system (section B of the figure), virtual machines (VM) typically package the application and operating system together. This allows for the appearance of multiple computing systems on a single physical device. In this case, the physical computer is referred to as the host, while the individual virtual machines are referred to as guests.

In a virtual machine, there is a software layer between the hardware and virtual machines. This is the virtualization management system, often referred to as a hypervisor. This can be implemented in one of two ways. First, it may exist as an operating system on its own, interfacing directly with the computing hardware. The alternative is for another software layer that sits above the operating system and interacts through the operating system.

  

**⇒ Comparing Virtualized Systems**

In terms of benefits, virtualized systems permit a physical server to run different operating systems, such as windows or iOS. These are referred to as operating system virtualization as described in the table **Types of Virtualization**. The table provides an overview of the different types of virtualization which can be implemented. A negative aspect of virtualization is the potential that the systems will run slower since the virtual machines’ access to the physical hardware must be scheduled by the hypervisor.

![[Screenshot_2023-02-02_at_5.53.42_PM.png]]

  

**⇒ Containers and Kubernetes**

Virtual machines are not the only approach to operating multiple applications on a physical machine. Another approach is called containers (yes, like those for shipping). In this case, the applications and key software resources, such as libraries, are isolated. Yet, the applications share the physical system’s operating system. The structure results in lower computing overhead. As a result, it is more efficient.

![[Screenshot_2023-02-02_at_5.54.44_PM.png]]

  

Containers greatly simplified the development and deployment of software. However, problems remain. Since only the application and related libraries are being packaged, many can be installed on a single host system. The issue that emerges is that the containerization software can only effectively manage a few containers at a time. However, much larger systems of servers may have to manage several hundred containers.

To deal with this problem, container management software, known as Kubernetes, was made open-source by Google. Kubernetes is used to manage containers on large-scale server systems. This allows for improved grouping, scheduling and load balancing in a large scale, container-based system.