In this chapter, we will see how to launch and manage a virtual machine on AWS. A virtual machine runs on a physical machine isolated from other virtual machines by the **hypervisor** — a software or hardware component that enables the creation and management of virtual machines.

- Th physical machine is called the **host machine**
- Virtual machines running on the physical machine are called **guests**
- Hypervisor isolates the guests from each other and schedules requests to the hardware by providing a virtual hardware platform to the guest system. The below is a good visualization:

![[Screenshot_2023-12-27_at_12.59.11_PM.png]]

Typical use case for a virtual machine follow:

- Hosting web application such as word press
- Operating enterprise application, such as ERP (Enterprise resource planning) application
- Transforming or analyzing data, such as encoding video files

  

**→ Example: Launching a virtual machine**

We’ll look at launching a virtual machine that runs a tool called “Link-checker”, that checks a website for broken links (links that may result in 404 Not Found errors). The infomration we enter to create one consists of:

- Naming the virtual machine
- Selecting the operating system — we will use AWS’s linux distribution for this
- Choosing the size of the virtual machine
- Configuring details
- Adding storage
- Configuring a firewall
- Granting the virtual machine permissions to access other AWS services