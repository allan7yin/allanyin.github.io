  

**Key Terms:**

- **Business Process Model:** a Business Process Model provides a representation of how an organization processes information or materials to achieve its objectives
- **Business Process Modelling and Notation (BPMN):** a method for representing business process models. It consists of a standardized set of symbols (the notation) that helps create model representations that can be understood by a wide range of users
- **Software Development Life Cycle:** a framework defining tasks performed at each step in the software development process.
- **Unified Modelling Language:** provides a set of standardized diagrams and symbols that assists in representing business processes for software development.

  

## 2a. The Phoenix Project

Related Article:

> [!info] Microsoft Issues Windows 10 'Black Screen' Update Warning  
> Spotted by the eagle eyes of Windows Latest, Microsoft has warned users that its new KB4503327 security upgrade can cause a 'Black Screen' .  
> [https://www.forbes.com/sites/gordonkelly/2019/06/18/microsoft-windows-10-update-upgrade-problem-warning-black-screen-boot-restart/?sh=ab63e034260e](https://www.forbes.com/sites/gordonkelly/2019/06/18/microsoft-windows-10-update-upgrade-problem-warning-black-screen-boot-restart/?sh=ab63e034260e)  

**Key Takeaways:**

- This chapter of the story highlights the importance of testing before moving software into production
- We also begin to see the amount of interaction between groups within an organization, and how communication needs to be clear
- Something that becomes apparent in this part of the story is the link between financial management oversight and Information Technology operations
- **High Touch Marketing:** When we speak of high touch marketing, we mean that the organization is very responsive to the needs and desires of its customers.
    
    - It also entails the use of technologies related to artificial intelligence and data analytics to understand purchasing behaviour and anticipate the customer’s needs, just as you find when you shop on Amazon or search on Google.
    - Will use CRM (Customer Relationship Management) System to maintain connection with the customer’s and to understand the customer’s needs better
    
      
    

## 2b. Business Process Modelling

**⇒ What is business process modelling**

- The objective of business process modelling is to improve the understanding of business operations so that better decisions can be made. It provides a basis for understanding and communicating the various interactions that take place within an organization as it works to deliver value to its customers.
- Process modelling allows for a quantitative and qualitative understanding of how the business operates and how information technologies align with these operations to achieve corporate objectives. In the story from the textbook, the initial ‘crisis’ concerned the payroll system. Let's examine a payroll process in an organization.

  

**⇒ Process Narratives and Storyboarding**

- One problem that often comes up when analyzing business processes is where to start. One approach is Process Narratives and Storyboarding. This may be useful when mapping a business process for the first time or undertaking a review to determine if changes have been made or are necessary.
- When developing business process models, we often start with a process narrative, which can then be turned into a storyboard. The process narrative is really just a way of getting people to describe what they do in plain language.

  

### Business Process Modelling

![[Screenshot_2023-01-26_at_4.23.34_PM.png]]

- **Square** is task
- **Diamond** is decision
- **Circle** is beginning and end
- **“+”** means there are sub-tasks

What is a business function is delegated to a third party?

![[Screenshot_2023-01-26_at_4.25.11_PM.png]]

### UML Diagrams

![[Screenshot_2023-01-26_at_4.27.55_PM.png]]

- Secondary actors assist the primary actor to complete the use case, in the above case, getting paid
- Actor can be any entity: human, organization, program, etc.

![[Screenshot_2023-01-26_at_4.32.45_PM.png]]

- Now, the above is the skeleton, but now labeled for the specific scenario. The middle bubbles represent tasks relevant to the system, and the arrows represent the relationship and direction of process flow in the system between the data points.
- The lines from the actors to the processes indicate interaction between the actors and the processes. Think of the lines from actors to the processes almost like “verbs”. The employee **completes** the employee information, HR staff **updates** the database, etc.
- The **include** relation lines indicate an include relationship, which means it is apart of the another process. For example, in the above, the user authorization process is part of the other processes.
- The **extend** relation indicates a process executed in the event of a failure, say, failure to authorize results in some security process being run.

  

## 2c. Models in Action

Now, here are some videos we refer to in order to obtain a better grasp on how to use these modelling tools.