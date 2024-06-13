## Dynamic Programming III

**⇒ Problem: Minimum Length Triangulation**

- **Input:**
    - $n$﻿ points $q_1, q_2, ..., q_n$﻿ in 2D space that form a **convex** **$n$**﻿-gon $P$﻿
        - Assume that these points are **sorted clockwise** around the centre of $P$﻿
- **Find:**
    - given the $n$﻿ points, _**triangulation**_ is the process of decomposing the $n$﻿-gon into a set of non-overlapping triangles
    - we want to find a triangulation of $P$﻿ such that the sum of the perimeters of the $n-2$﻿ triangles is minimized
- **Output:**
    - the **sum** of the **perimeters** of the triangles in $P$﻿

  

So, given this $n$﻿-gon, how many possible triangulations are there — there are (n-2) Catalan number many, which is: $C_{n-2} = \frac{1}{n-1}\binom{2n-4}{n-2}$﻿. We don’t need to know how, but $C_{n-2} \in \Theta(\frac{4^n}{(n-2)^{\frac{3}{2}}})$﻿

  

come back to this