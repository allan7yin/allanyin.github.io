### ResponseEntity<\?>

- When responding to something over http protocol, look to respond with a `ResponseEntity<>` object, which wraps around some other data type. It is a class that represents the HTTP response returned by a controller method. It encapsulates the response status, headers, and body, allowing you to customize and control the HTTP response returned to the client.
    - Here are popular use cases:

```Java
@GetMapping("/data")
public ResponseEntity<String> getData() {
    String data = "This is the response data.";
    return ResponseEntity.ok(data); // 200
}
```

```Java
// In another method or class
ResponseEntity<String> responseEntity = getData();
String responseData = responseEntity.getBody(); // can use hasBody() to check first too
```

In short, it wraps whatever data we want to return in a `ResponseEntity` object, which can then pass additional parameter such as `HTTP.STATUS_CODE` and header data.

In addition, we can use these with **wildcards**. The <\?> part is a wildcard, which means that we can expect any type. This means we can return any type within the `ResponseEntity`, and it will be fine.