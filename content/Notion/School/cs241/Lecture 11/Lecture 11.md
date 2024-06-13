We continue looking at how to construct a predict table

**⇒ Helper Function: Follow()**

To understand `Follow()`, we need to add a rule to our original grammar where a non-terminal derives the empty string, e.g. rule 7: B→ Empty String.

![[Screenshot_2023-03-07_at_2.13.48_PM.png]]

Now, we can get something like S ⇒ **⊢**S**⊣ ⇒** ⊢AyB⊣ ⇒ ⊢abyB⊣ ⇒ ⊢aby⊣.

- ⊣ can appear after B, but there is no derivation from B, that leads to ⊣. We conclude that ⊣ is the follow set of B

Refer to slides for this one:

![[CS241-Slides_312-332.pdf]]