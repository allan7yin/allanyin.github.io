## Section 4,5:

Now, to begin learning other new things, we begin a simple “messages” application. We will simply retrieve and post data to a static JSON file.

![[Screen_Shot_2022-10-13_at_7.32.15_PM.png]]

  

![[Screen_Shot_2022-10-13_at_7.39.20_PM.png]]

  

![[Screen_Shot_2022-10-13_at_7.39.38_PM.png]]

![[Screen_Shot_2022-10-13_at_7.44.26_PM.png]]

So, this above chunk is a visualization of what the finished product will look like.

  

---

  

To begin, it is useful to make use of Nest.js CLI to make creating things much faster.

```Bash
nest generate module messages

nest generate controller  messages/messages --flat
```

  

![[Screen_Shot_2022-10-13_at_8.30.40_PM.png]]

![[Screen_Shot_2022-10-13_at_8.35.10_PM.png]]

Once we have set up the request handlers, we can simply test using either POSTMAN, or a VSCode extension: REST client. Then, we make the file, something like, “requests.http”, and populate it with:

```Bash
### THIS IS ALTERNATIVE TO POSTMAN, PERSONALLY I LIKE IT BETTER 
### TESTING REQUEST HANDLERS 

### List all messages 
GET http://localhost:3000/messages


### Create a new message 
POST http://localhost:3000/messages
content-type: application/json

{
    "content": "hi there"
}


### Get a particular message
GET http://localhost:3000/messages/328299581
```

![[Screen_Shot_2022-10-13_at_8.56.56_PM.png]]

So, with these decorators, we can parse the URL to obtain what we want. This can be the query string, a specific parameter, or the body of the response, as shown above.

  

So, at this point, we have created a controller. Now, say we want to validate data that is being posted. We use a pip for this purpose.

![[Screen_Shot_2022-10-14_at_10.15.08_AM.png]]

![[Screen_Shot_2022-10-14_at_10.15.58_AM.png]]

ValidationPipe is the built-in Nest pipe to validate things. To create a validation pipe, we simply:

```Bash
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { MessagesModule } from './messages/messages.module';

async function bootstrap() {
  const app = await NestFactory.create(MessagesModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

  

Now, we need to create a special set of rules for the validation pipe to follow.

![[Screen_Shot_2022-10-14_at_10.20.20_AM.png]]

We need only to to step 1, once, which is the code above. For every thing we want to validate, we will need to repeat steps 2,3,4.

  

In particular, for step 2, the class we make is called a **Data Transfer Object (dto)**. This is what you’ve been seeing in the company back-end API repo. So, here is to code to make sure that the content of a POST request is a string:

```TypeScript
// in create-message.dto.ts
import { IsString } from 'class-validator';

export class createMessageDto {
    @IsString() // this is a validator. We can now ensure that whenever we make an instance of createMessageDto, that the content is a string 
    content: string;
}

// in messages.controller.ts
@Post()
    createMessage(@Body() body: createMessageDto) { // here, Nest extracts the body for us, and supplies it as a parameter for us
        console.log(body);
    }

// note how now, we extract the body and say it is of type createMessageDto,
// my guess is it takes the string and implicitly calls the constructor of 
// createMessageDto
```

![[Screen_Shot_2022-10-14_at_1.54.14_PM.png]]

The diagram above illustrates what is happening here. Recall that we needed to `npm install class-transformer class-validator`. These are what power this process. First, the JSON is converted into an instance of the DTO class (in our case, the CreateMessageDto). Then, after we have a DTO object, the decorators run to run validation with class-validator. Lastly, it checks if there are errors.

  

![[Screen_Shot_2022-10-14_at_2.13.54_PM.png]]

  

![[Screen_Shot_2022-10-14_at_2.13.41_PM.png]]

So, let’s see how to implement the repository and the service. Code is simple, can be found in the messages folder.

---

  

Now, one of the most important concepts about Nest.js is dependency injection. While it is very easy to briefly go over the syntax and how to use it, it is imperative we understand what it is trying to accomplish.

  

To begin, we go over some important concepts.

### Inversion of Control Principle

This principle states: Classes should not create instances of its dependencies on its own. Earlier, we said that this:

```TypeScript
constructor() {
        // we have now setup a dependency between the two classes , WE DON'T ACTUALLY DO THIS. 
        // There is something called dependency injection, so the line below will soon be obsolete
        this.messagesRepo = new MessagesRepository();
    }
```

was a terrible. Instead, what is the best way of doing this is:

  

![[Screen_Shot_2022-10-14_at_5.00.12_PM.png]]

  

![[Screen_Shot_2022-10-14_at_5.00.31_PM.png]]

  

However, this principle also comes with its downsides.

![[Screen_Shot_2022-10-14_at_5.06.08_PM.png]]

With this, it is very lengthy. We now need 3 lines to create one controller as opposed to the one line for the original code. This may multiply by many times when we add more services and controllers. To solve this problem, we introduce **dependency injection**

  

### Dependency Injection

To solve the above problem and maintain inversion of control, we use this.

![[Screen_Shot_2022-10-14_at_5.12.28_PM.png]]

![[Screen_Shot_2022-10-14_at_5.12.53_PM.png]]

  
So, now that we know how the internals of dependency injection work, let’s refactor out code to use it. To see this, visit the project folder.