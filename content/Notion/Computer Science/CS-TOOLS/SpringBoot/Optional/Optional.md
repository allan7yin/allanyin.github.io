### Optional<\?>

The `Optional<>` is a type that we can use when we are not sure a function will return anything. Consider something like:

```Java
Optional<MqResponse> response = mqResponseRepository.findById(correlationId);
```

We don't know if it will find anything, and it won't throw an error if it does not.

Optional is a container object which may or may not contain a null value. If the value is present, we can determine this with `isPresent()` and retrieve it with `.get()`.