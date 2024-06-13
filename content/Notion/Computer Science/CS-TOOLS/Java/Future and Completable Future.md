### Future<\?>

`Future` objects are how we work with asynchronous functions in Java. To quickly recap on what asynchronous programming means:

- It is a means of writing non-blocking code by running a task on a separate thread than the main application thread, and notifying the main thread about its progress, completion, or failure.

### Future

A Future is used as a reference to the result of an asynchronous computation. It provides an `isDone()` method to check whether the computation is done or not, and a `get()` method to retrieve the result of the computation when it is done. This is a good step towards asynchronous programming in Java, but had certain limitations.

- **Cannot be manually updated:** We cannot manually complete a future object. Say we're waiting for some method on another thread to complete. But, after a while, we decide we don't need it and we'll use some other saved data. With Future, we cannot manually complete this, and will continue to block.
- You cannot perform further action on a Future’s result without blocking:
    - Future does not notify of completion, can only `.get()`, which blocks until result is available

There are more, but we won't go into them. Because of these limitations, Java has a `CompletableFuture`. With this, we can manually complete these if needed. Here are some trivial examples:

```Plain
CompletableFuture<String> completableFuture = new CompletableFuture<String>();

String result = completableFuture.get()

completableFuture.complete("Future's Result")
```

If you want to run some background task asynchronously and don’t want to return anything from the task, then you can use `CompletableFuture.runAsync()` method. It takes a `Runnable` object and returns `CompletableFuture<Void>`:

```Plain
// Run a task specified by a Runnable Object asynchronously.
CompletableFuture<Void> future = CompletableFuture.runAsync(new Runnable() {
    @Override
    public void run() {
        // Simulate a long-running Job
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
        System.out.println("I'll run in a separate thread than the main thread.");
    }
});

// Block and wait for the future to complete
future.get()
```

There are many more features, but these are the fundamentals that you need to know. For reference, here is how it was used in QuizGpt:

```Plain
private CompletableFuture<String> GetResponseOrWait(String correlationId) throws TimeoutException, CorrelationIdNotFound, InterruptedException {
        long startTime = System.currentTimeMillis();
        long timeout = 5000; // Timeout in milliseconds

        CompletableFuture<String> futureResponse = new CompletableFuture<>();
        MqResponse response = null;

        while (System.currentTimeMillis() - startTime < timeout) {
            // Check if entry is present in the database table
            boolean entryExists = checkEntryInTable(correlationId);

            if (entryExists) {
                response = accountService.FindMqResponseByCorelationId(correlationId);
                logger.info(response.getResponse());
                futureResponse.complete(response.getResponse().replace("response=", ""));
                accountService.MqDelete(response);
                return futureResponse;
            }

            // Wait for 100 milliseconds before checking again
            Thread.sleep(100);
        }

        throw  new TimeoutException("Timeout: Took too long to fetch (> 5seconds) " + correlationId);
    }
```