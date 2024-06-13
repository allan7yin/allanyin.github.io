### 1a.

20949813

### 1b.

```C
> sf <- subset(mydata, subset = (city == "sf"))
> sf$subject.race <- factor(sf$subject.race, levels = c("asian/pacific islander", "black", "hispanic", "white", "other"))

> sexRaceTable <- table(sf$subject.race, sf$subject.sex)
> prop.table(sexRaceTable, 2)
```