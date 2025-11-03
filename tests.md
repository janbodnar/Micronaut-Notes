# Tests

## Test method not allowed

```java
@Test
void testPatternGetMethodNotAllowed(@Client("/") HttpClient httpClient) {

    BlockingHttpClient client = httpClient.toBlocking();
    assertThrows(HttpClientResponseException.class,
            () -> client.exchange(HttpRequest.GET("/some-path")),
            HttpStatus.METHOD_NOT_ALLOWED.getReason());
}
```

## @MicronautTest

`@MicronautTest` integrates Micronaut's dependency injection, configuration, and application 
context into your test classes, making it easy to write tests that interact with 
your Micronaut application.

```java
package com.example;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import io.micronaut.context.ApplicationContext;

class MessageServiceTest {

  @Test
  void applicationContextWithoutMicronautTest() {
    try (ApplicationContext context = ApplicationContext.run()) {
      MessageService messageService = context.getBean(MessageService.class);
      Assertions.assertEquals("Hello World", messageService.message());
    }
  }
}
```

Simplified with `@MicronautTest`:

```java
package com.example;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;

@MicronautTest(startApplication = false)
class MessageServiceTest {

  @Test
  void applicationContextWithoutMicronautTest(MessageService messageService) {
    Assertions.assertEquals("Hello World", messageService.message());
  }
}
```
