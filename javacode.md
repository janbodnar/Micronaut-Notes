# Java code

## Test Not Found exception

```java
@Test
void testHelloFilterNotLogged(@Client("/") HttpClient httpClient) {
    
    BlockingHttpClient client = httpClient.toBlocking();
    assertThrows(HttpClientResponseException.class,
            () -> client.retrieve(HttpRequest.GET("/nothello")),
            HttpStatus.NOT_FOUND.getReason());
}
```
