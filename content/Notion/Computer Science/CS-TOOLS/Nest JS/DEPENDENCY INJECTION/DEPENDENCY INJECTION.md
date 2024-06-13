⇒ **Note: This file serves to solely recap on how dependency injection works, and the principle that propels it**

![[Screen_Shot_2022-10-23_at_8.34.09_PM.png]]

Consider the diagram above. We can see clearly the dependencies that exist with the entire application. The service module relies upon the repository, for example, and the controller directly relies upon the service, and indirectly relies upon the repository. Consider the following code:

```TypeScript
export class MessagesService {
	messagesRepo: MessagesRepository;
	
	constructor() {
		this.messagesRepo = new MessagesRepository();
	}

	findOne(id: string) {
		return this.messagesRepo.findOne(id);
	}
// and the rest of the class for this would go below here
}
```

Now, above is a short snippet of the service module file class. Above, as we `new` an object inside of the `MessagesService` class, the class is creating its own dependencies. This brings us to the the **inversion of control** principle.

## Inversion of Control

⇒ _Classes should not create instances of its dependencies on its own_

Here is the above code, refactored to follow the inversion of control principle:

```TypeScript
interface Respository {
	findOne(id: string);
	findAll();
	create(content: string);
}

export class MessagesService {
	messagesRepo: Repository;
	
	constructor(repo: Repository) {
		this.messagesRepo = repo;
	}
}
```

Now, let’s take a look at why this would be vastly better than the version above. To begin, we have defined a **Repository** interface, which holds the functions that we wish to implement, and should be inside the repository. By doing this, our code not only works with the MessagesRepository, but any object that satisfies the interface will work with our code. But why is this way of desiging our code a good idea? The main points can be summarized here:

![[Screen_Shot_2022-10-23_at_9.30.51_PM.png]]

Now, inversion of control does not come without its faults. One of the major downsides of using it, is the quantity code of required for some task greatly increases. To solve this issue, we now introduce **dependency injection**, and look at how it works.

![[Screen_Shot_2022-10-23_at_10.31.40_PM.png]]

Every time we make a class in a Nest.js application, if its not the controller or module (so really only the service and the repository), it will be added to the Nest.js DI container. In there, each class is mapped to its dependencies. So, when we go and try to make a controller, Nest will look at the constructor, see we want an instance of the `MessagesService`, and create the dependencies for us. Notice that with DI, we can’t do the inversion of control things from before, which is fine though.

![[Screen_Shot_2022-10-23_at_10.35.04_PM.png]]