### Invariance and Equivariance

> [!info] 05 Imperial's Deep learning course: Equivariance and Invariance  
> Admin about this course: http://wp.  
> [https://www.youtube.com/watch?v=a4Quhf9NhMY&ab_channel=BernhardKainz](https://www.youtube.com/watch?v=a4Quhf9NhMY&ab_channel=BernhardKainz)  

These 2 are important concepts in **convnets.**

**→ Shift Invariant**

This concept is very important in image recognition we have seen so far. Consider the following images:

![[Screenshot_2024-02-05_at_10.50.16_PM.png]]

_**Which of the following pictures has a cat inside?**_ Of course, both are cats. This property is known as shift invariance, and is important for images to have this to be classified using convnets.

  

**→ Shift Equivariant**

Shift equivariance means the output shifts in the same way as the input → **shift in output mirrors shift in input**

![[Screenshot_2024-02-06_at_9.48.14_AM.png]]

What is the context of this in **convnets?**

![[Screenshot_2024-02-06_at_9.53.56_AM.png]]

**→ Convolutions are shift-equivariant operations**

Consider this:

![[Screenshot_2024-02-06_at_9.54.41_AM.png]]

No matter where I shift that **black 3** in the left box, after a convolution with that **3-colour kernel**, the **coloured-3** will always mirror the shift of the black 3 → hence this operation is **shift-equivariant**

  

**→ Pooling is shift-invariant (approximate)**

Consider this:

![[Screenshot_2024-02-06_at_10.34.11_AM.png]]

In max-pooling, we are taking the maximum value within that kernel. So, even if the black 3 slightly shifts, the result of max-pooling does not change → hence this operation is **shift-invariant**