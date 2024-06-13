Now, we move on to something new.

  

### Section 6: Nest.js Architecture: Organizing Code with Modules

  

To help learn why we use modules, we will do so through a small demonstartion. Suppose we want to make an application that models a computer, as shown:

  

![[Screen_Shot_2022-10-15_at_10.27.38_AM.png]]

So, to generate these files above, we simply use the terminal:

```TypeScript
nest g module computer
// it always goes like that, nest, g (for generate), what we want (controller, service, repo, module, etc.) and what class (computer, disk, CPU, power, etc.)
```

  

After we have generated the modules, we can start connecting them in the way they are intended.

  

![[Screen_Shot_2022-10-15_at_1.44.01_PM.png]]

![[Screen_Shot_2022-10-15_at_1.44.44_PM.png]]

To see how these are implemented in the code, visit the project folder to see how.