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
import io.micronaut.runtime.server.EmbeddedServer;
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;

@MicronautTest(startApplication = false)
class MessageServiceTest {

  @Test
  void applicationContextWithoutMicronautTest(MessageService messageService) {
    Assertions.assertEquals("Hello there!", messageService.message());
  }

  @Test
  void embeddedServerNotRunning(EmbeddedServer embeddedServer) {
    Assertions.assertFalse(embeddedServer.isRunning());
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
      Assertions.assertEquals("Hello there!", messageService.message());
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
            {"message":"Hello there!"}""", json);
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
        {"message":"Hello there!"}""", json);
  }
}
```

## Test serialization

The classes to be serialized bust be annotated with `@Serdeable` annotation. 

```java
package com.example;

import io.micronaut.serde.annotation.Serdeable;

@Serdeable
public record Post(Long id, Long userId, String title, String body) {
}
```

`JsonMapper` is a Micronaut interface for serializing and deserializing JSON.  
It provides methods like `readValue` (to convert JSON strings to Java objects)  
and `writeValueAsString` (to convert Java objects to JSON strings)

```java
package com.example;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import io.micronaut.json.JsonMapper;
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;

@MicronautTest(startApplication = false)
class PostSerializeTest {

  @Test
  void deserializePost(JsonMapper jsonMapper) throws Exception {
    String json = """
        {
          "id": 1,
          "userId": 11,
          "title": "Title 1",
          "body": "Body 1"
        }
        """;

    Post post = jsonMapper.readValue(json, Post.class);

    Assertions.assertEquals(1L, post.id());
    Assertions.assertEquals(11L, post.userId());
    Assertions.assertTrue(post.title().startsWith("Title 1"));
    Assertions.assertTrue(post.body().contains("Body"));
  }

  @Test
  void serializePost(JsonMapper jsonMapper) throws Exception {
    Post post = new Post(1L, 11L, "Title 1", "Body 1");
    String json = jsonMapper.writeValueAsString(post);

    Assertions.assertTrue(json.contains("\"id\":1"));
    Assertions.assertTrue(json.contains("\"userId\":11"));
    Assertions.assertTrue(json.contains("\"title\":\"Title 1\""));
    Assertions.assertTrue(json.contains("\"body\":\"Body 1\""));
  }
}
```


