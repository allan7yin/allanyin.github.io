## Sections 1,2,3

```JavaScript
npm install @nestjs/common@7.6.17 @nestjs/core@7.6.17 @nestjs/platform-express@7.6.17 reflect-metadata@0.1.13 typescript@4.3.2
```

  

We can install these dependencies manually by using the terminal:

![[Screen_Shot_2022-10-13_at_3.39.13_PM.png]]

  

But this is redundant. We only do this to show what happens when we set up a Nest project. Normally, this is all completed by default through the CLI.

![[Screen_Shot_2022-10-13_at_3.43.52_PM.png]]

![[Screen_Shot_2022-10-13_at_3.44.09_PM.png]]

![[Screen_Shot_2022-10-13_at_3.46.13_PM.png]]

  

![[Screen_Shot_2022-10-13_at_3.51.11_PM.png]]

  

So, let’s first start with creating the controller and module. Controller and Module are things that NestJS provide us, and so, to create them, we simply do:

```TypeScript
import { Controller, Module, Get } from '@nestjs/common'; // 95% of this we need to import from Nest js will be from common 
import { NestFactory } from '@nestjs/core';

@Controller() // a decorator, telling nest we are trying to create a class that will serve as a controller in our application 
class AppController {
    @Get()
    getRootRout() {
        return 'hi there!';
    }
}

@Module({ // need to pass in configurations to this decorator 
    controllers: [AppController]
})
class AppModule {}

async function bootstrap() {
    const app = await NestFactory.create(AppModule); // this creates a Nest application with the AppModule 
    await app.listen(3000); // if we make a get request to localhost:3000 with no route on it, it is directed to route handler above (which is the Get decorator)
}

bootstrap();
```

![[Screen_Shot_2022-10-13_at_5.31.14_PM.png]]

The above a diagram is what we do to separate the different things into separate files. There, is the simple code needed to make a Nest application.

---

  

Now, the above stuff is good to know, but we will practically never start a project like that. Instead, we will need