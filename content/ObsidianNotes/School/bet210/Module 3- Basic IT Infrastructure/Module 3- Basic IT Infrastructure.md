## Introduction

In this module, we’ll begin to look at the various technologies which can be found within an organization’s Information System. This is referred to as an organization’s information technology (IT) infrastructure.

### Key Terms

- **Client:** A client is the receiving end of a service or the requester of a service in a client/server model type of system. The client is most often located on another system or computer, which can be accessed via a network ("What is a Client?", n.d.).
- **Network:** A network is a system of two or more computers linked together to permit them to share resources and/or information.
- **Operating System:** The operating system is the software that manages the computer’s resources, such as memory allocation, CPU scheduling, Input/Output (I/O) devices, among various other programs.
- **Server:** A server is a computer which provides information and/or other services, such as printing, to multiple users called clients.

  

## 3a. The Phoenix Project

There are many types of work within an organization. But, the term “work” is rather broad, and doesn’t really have a concrete definition. In the book, at this point, three of the four types of work found within an organization have been seen:

1. Business projects
2. Internal IT projects
3. Changes

## 3b. Introduction to Information Technology Hardware

1. **Personal Computing:** Consider the following devices:
    
    ![[Screenshot_2023-02-02_at_4.30.47_PM.png]]
    
2. **Adding a Router:** To connect these devices together, we use intermediary devices, such as the router, and others. Connecting to external things needs middle-man devices. Everything is well, until you want to access Netflix, or, access something like UW’s Learn system.
    
    ![[Screenshot_2023-02-02_at_4.32.08_PM.png]]
    
3. **Connecting to ISP:** To address this, we need to connect to the outside world. This is done through a modem. We will assume the router and modem are separate devices, although sometimes, they are combined into one entity. There are different modems that serve different purposes:
    
    1. If you are connecting to the existing telephone system, you would likely use an ADSL modem (Asymmetric Digital Subscriber Line). The term asymmetric simply means that the download speed is higher than the upload speed (the bandwidths specified in megabytes per second or Mbps). This comes in handy if you want to stream your favourite show while tweeting at the same time.
    2. Other forms of models can be “cable modems”, which would connect to the coaxial cable for your television instead of the telephone system, or fibre optic which would use light transmission over a cable for communications.
    
    The information sent from the modem goes through the provider’s network and eventually arrives at your internet service provider (ISP, something like Bell, Rogers, etc.). They take your connection, whether through cable or telephone lines, and provide the interface to services such as DNS servers and the high-speed communications networks which comprise the internet.
    
    ![[Screenshot_2023-02-02_at_4.42.15_PM.png]]
    
4. **Connecting to the server:** Once the information is received at the ISP, the web address that you specified, the URL, is found by checking a Domain (DNS) Server. The DNS looks up the numerical Internet Protocol (IP) address. This is essentially looking up the name, written in a form which can be understood by people, in a list and converting it to numbers, such as 123.456.789.012 which can be understood by computer hardware. Once the IP address has been determined, your transmission is routed to the web service you are interested in connecting to, such as Netflix.
    
    ![[Screenshot_2023-02-02_at_4.43.43_PM.png]]
    
      
    
    In the previous example, we showed a single server. In many cases, this is not realistic because the number of clients that can connect to a server can be large and a single server would not be able to handle the traffic. The number of clients that a server can handle will be a function of the amount of memory (Random Access Memory or RAM) available and the characteristics of the central processing unit (CPU).
    
    These characteristics include the speed of the CPU in GHz and the number of cores (i.e: 2, 4, 8, 36). One way to think about the cores in a CPU is as processing units that can independently run code. The code being executed on a core are referred to as threads.
    
    When an application is running on a computer it will have at least one thread if a single core is processing only that application one single command at a time. However, in a multiple core CPU, a program may generate more than one thread, so it is being executed simultaneously on more than one core. This results in faster program execution.
    
    In the case of a system which has many users on it at one time, this becomes important. The number of user requests which can be handled by a single server is determined by the characteristics of the CPU, the amount of memory available per core, and the performance of the server software. To reliably deal with a high traffic environment, IT operations may create a “data centre” or “server farm” consisting of many servers, perhaps in the hundreds or thousands. Large data centres can require millions of square feet of space and have very high power and cooling requirements.
    
    ![[Screenshot_2023-02-02_at_4.46.14_PM.png]]
    

## 3c. Exploring IT Software

We’ve considered the hardware that can be found in a simple network. Now, let’s explore what is going on inside of the computing hardware discussed in the previous section. Let’s begin with the client.

  

**⇒ The Client**

To start, the computer will consist of a main circuit board, generally referred to as a “motherboard”. A large number of hardware components are found in these boards, such as the central processing unit (CPU), random access memory (RAM) and different input/output controllers that interface to things like the mouse, keyboard, monitor, and communications. All other devices in the computer, such as the hard drive(s) will connect to this board.

  

If you were to look at the “Device Manager” in a PC computer, you might see something similar to the screenshot below. This gives you an idea of how many types of components can be found in a computing device. In this example, each of the headings could be expanded to look at individual devices. For example, expanding the disk drive section may indicate multiple hard drives in the system, while expanding the processors section would show the 8 cores in this computer CPU. The problem now becomes how do you, the user, interact with all these components?

  

![[Screenshot_2023-02-02_at_5.24.57_PM.png]]

  

To allow the user to interact with all these hardware components, we need to develop a software layer: the operating system. Windows 10 is an example of this software.

The operating system manages the various system resources so that they are accessible by other programs (applications) that the user can use. It consists of the basic input output system (BIOS) which allows the hardware to load in programs like Windows 10. There are also various drivers which allow the software to control the components such as monitors, hard drives and network communications.

Now that we have the operating system installed and working properly, we can install other software for the user. The type of software installed will depend on the needs of the end user. For a typical user, this may consist of applications such as Microsoft Office. This will contain various libraries and other software components that other applications will need to run properly. Ultimately, you and other users can use applications such as Word or Excel to do your work without caring what is going on at the operating system level.

![[Screenshot_2023-02-02_at_5.25.25_PM.png]]

  

**⇒ The Server**

We’ve examined the software side of the client computer, but what if you want to go to your favorite website, such as Waterloo Learn. On your computer, you (the client) would open an application, i.e., your browser (Chrome, Firefox, Edge, etc.). These would communicate with the outside world via the operating system and hardware we discussed in the previous section.

![[Screenshot_2023-02-02_at_5.28.29_PM.png]]

Once connected with the desired server hardware, more software comes into the picture. There may be different types of hardware components, such as different CPUs and memory, but in many ways this is similar to the system we examined in your home computer. There is an operating system which manages the interfaces to the hardware, as well as the communication, and control aspects of managing several clients at the same time. There can be different types of server roles, such as a web server.

What we are connecting to in this example may include email servers which provide email services to many end users, or file servers, which provide users with access to information. File servers are similar to what you would use if you subscribe to ‘Dropbox’. So when you do something through your browser, a message is created and transmitted to the server. The server then acts on the message and transmits the response back to your browser. You see this as a modification of the information presented by your browser.

While this may seem like a simple thing to do, the system of networks that we call the internet, and the associated hardware and software is very complex and constantly evolving. So next time you sit back to watch Netflix on your laptop or phone, consider how awesome it is that you can actually do that.