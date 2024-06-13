This page will house all needed information I should know — to fully explain what I have done in my last co-op as well as the tools surrounding it

  

- Worked at Bank of America Merrill Lynch — global market data management team
- During my duration here, primarily worked on 2 projects, along with other tasks from other teams occasionally
- first month — individuals on my immediate team were on vacation, manager was gone for a month. In this case, there were no tasks being assigned me. Took it upon myself to find something to work on. Contacted developers on other team — risk management team and mortgages team. They had an application, which was outsource, that connected us and some large corporate clients, which had loans from us. The teams had built code around that application that extracted certain information from that applications, and sent that data to other boa teams.
    - what did I do? I continued unfinished Java programs that extracted specified information from the application, such as borrower information, credit report, financial statements, loan purpose, load amount and time, etc.
    - for some people, they would then be given a score: how likely they are to continue their mortgage/whether we should extend their contract duration
    - for others, if assessed as risk by risk team’s program, would mark those entities as “risk”
    - final data is then passed to other teams in other departments
- Updated other Java applications — there are 5 main applications in the company, each one being built on somewhat old Java code
    - often, each file would have tens of files that utilized data classes with hundreds of lines of getters/setters/serialization functions
    - at the same time, was talking with some other new employees, found it annoying for there to be so much boilerplate code
    - I suggested the use of Lombok, discussed its benefits and drawbacks with peers
    - Integrated it into all boa Canada projects for the trading department → easily 10k lines changed
    - pushed into production level code at the end of the co-op term
    - no change in compile time — greatly increased in readability, maintainability
- when team back, worked on lower level bash scripts that served multiple functions
    - SFTP
- team was going through some linux migration changes, I needed to change various types of data, such as wideIP, mentioned in many files. Wrote some scripts that would automate this process, and placed them into server for other developers to use in the future if needed
- worked with equities team from both Toronto and New York — trade data being sent to New York was buggy (they had just updated their system). Multiple components of an equity trade all contained same trade id (which is fine in equities), but analysts in New York wanted only one
- designed Figma flow charts proposing my data aggregation ideas →
    - do via bash, concatenate into rows
    - create new tables in database, and create one-to-many relations between tables for the other teams to access.
    - Added new Java code to the existing infrastructure
    - Pushed to production
- Last month → back to mortgage backed securities. The application that was outsourced was now beginning to be developed by ourselves: company did not want to purchase another one as contract was expiring.
- Part of small team developing first steps of new springboot application. Created classes that provided functionality to bundle loans together as one package item. As the application is focused on selling mortgages, common practice was to bundle loan with others to form a bundled mortgage
- The application also was supposed to receive end of day trade data from traders from various locations. Team sending data also had teams creating new application, decided to try rabbitMQ as message broker. Learned working with RabbitMQ, and implemented a message listener that consumed from a couple queues.
    - To do this, used RabbitTemplate
        
        - Create connection with `AmqpTemplate`
        
        ```Java
        @Bean
            public AmqpTemplate amqpTemplate(ConnectionFactory connectionFactory){
                RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
                rabbitTemplate.setMessageConverter(jsonMessageConverter());
                return rabbitTemplate;
            }
        ```
        
        - declare the queue again and declare the exchanges and channels
        - create message listener
- Utilized Spring Security to create Authentication Filter’s for the application’s security filter chain
    - before I left, creatd basic JWT based authenitcation filter onto the existing auehtnication filters writte by team