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
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;

@MicronautTest(startApplication = false)
class MessageServiceTest {

  @Test
  void applicationContextWithoutMicronautTest(MessageService messageService) {
    Assertions.assertEquals("Hello World", messageService.message());
  }
}
```

The `startApplication (boolean, default: true)` determines whether to start the embedded  
application (e.g., HTTP server). Set to `false` for faster, server-less tests.

Without the annotation, we need to setup `ApplicationContext` manually.

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

## Tests with EmbeddedServer

Manual setup:

```java
package com.example;

import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

import io.micronaut.context.ApplicationContext;
import io.micronaut.http.client.BlockingHttpClient;
import io.micronaut.http.client.HttpClient;
import io.micronaut.runtime.server.EmbeddedServer;

class GreetingControllerTest {

  @Test
  void applicationContextWithoutMicronautTest() {
    try (EmbeddedServer server = ApplicationContext.run(EmbeddedServer.class)) {
      try (HttpClient httpClient = server.getApplicationContext().createBean(HttpClient.class, server.getURL())) {
        BlockingHttpClient client = httpClient.toBlocking();

        String json = assertDoesNotThrow(() -> client.retrieve("/"));
        assertEquals("""
            {"message":"Hello World"}""", json);
      }
    }
  }
}
```

With `@MicronautTest`:

```java
package com.example;

import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

import io.micronaut.http.client.BlockingHttpClient;
import io.micronaut.http.client.HttpClient;
import io.micronaut.http.client.annotation.Client;
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;

@MicronautTest
class GreetingControllerTest {

  @Test
  void applicationContextWithoutMicronautTest(@Client("/") HttpClient httpClient) {
    BlockingHttpClient client = httpClient.toBlocking();

    String json = assertDoesNotThrow(() -> client.retrieve("/"));
    assertEquals("""
        {"message":"Hello World"}""", json);
  }
}
```

