## Noted Things from Practice Questions

**Semaphore vs Mutex**

1. Semaphore does not have owner — maintains a count, whereas Mutex has owner
2. Semaphore can have 1→ n threads accessing, Mutex can only have 1
3. Semaphore can be initializd to anything 1→ n, Mutex needs to start as unlocked