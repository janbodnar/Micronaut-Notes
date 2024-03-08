# Tests

## Test method not allowed

```java
@Test
void testPatternGetMethodNotAllowed(@Client("/") HttpClient httpClient) {

    BlockingHttpClient client = httpClient.toBlocking();
    assertThrows(HttpClientResponseException.class,
            () -> client.exchange(HttpRequest.GET("/pattern-m")),
            HttpStatus.METHOD_NOT_ALLOWED.getReason());

    mlog.stopAppender();
}
```
