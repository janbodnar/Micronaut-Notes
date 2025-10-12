# Controllers


## Multiple URIs
 
```java
@Get(uris = { "/pattern", "/pattern1", "/pattern2", "/pattern3",
        "/pattern4", "/pattern5" }, produces = "text/plain")
String pattern() {

    return "calling pattern filters";
}
```

## Plain text

```java
@Get(value = "/status", produces = MediaType.TEXT_PLAIN)
public HttpResponse<String> status() {
    return HttpResponse.ok("text")
            .contentType(MediaType.TEXT_PLAIN);
}
```

## Status

```java
@Get("/")
@Status(HttpStatus.OK)
public String index() {
    return "Home page";
}
```


## Send header

```java
@Head(uri = "/pattern-m")
HttpResponse<?> methods() {

    return HttpResponse.ok()
            .header("X-TEST-HEAD", "HEAD request processed");
}
```

## Testing controller

```groovy
package testex.controller

import io.micronaut.http.annotation.Controller
import io.micronaut.http.annotation.Get
import io.micronaut.http.HttpResponse

@Controller
class MyController {

    @Get(uri='/hello', produces = 'text/plain')
    String hello() {

        return 'hello there!'
    }

    @Get(uri='/', produces = 'text/html')
    String home() {

        return HttpResponse.ok('home page')
    }

    @Get(uri='/status', produces = 'text/plain')
    HttpResponse<?> status() {

        return HttpResponse.ok()
    }
}
```

Testing http status, text body, exceptions  

```groovy
package testex

import io.micronaut.http.HttpRequest
import io.micronaut.http.HttpResponse
import io.micronaut.http.HttpStatus
import io.micronaut.http.client.HttpClient
import io.micronaut.http.client.annotation.Client
import io.micronaut.http.client.exceptions.HttpClientResponseException
import io.micronaut.runtime.server.EmbeddedServer
import io.micronaut.test.extensions.spock.annotation.MicronautTest
import jakarta.inject.Inject
import spock.lang.AutoCleanup
import spock.lang.Shared
import spock.lang.Specification

@MicronautTest
class MyControllerSpec extends Specification {

    @Shared @Inject
    EmbeddedServer embeddedServer

    @Shared @AutoCleanup @Inject @Client('/')
    HttpClient client

    void 'test home'() {
        when:
        client.toBlocking().exchange('/')

        then:
        noExceptionThrown()
    }

    void 'test hello endpoint'() {
        given:
        HttpResponse response = client.toBlocking().exchange('/hello')

        expect:
        response.status == HttpStatus.OK
        response.getBody(String.class).get() == 'hello there!'
    }

    void 'get endpoint with post method should fail'() {
        when:
        client.toBlocking().exchange(HttpRequest.POST('/', [msg: 'hello']))

        then:
        HttpClientResponseException ex = thrown()

        expect:
        ex.status == HttpStatus.METHOD_NOT_ALLOWED
    }

    void 'test status'() {
        given:
        HttpResponse response = client.toBlocking().exchange('/status')

        expect:
        response.status == HttpStatus.OK
    }

    void 'test non-existing endpoint'() {
        when:
        client.toBlocking().exchange('/notfound')

        then:
        HttpClientResponseException ex = thrown()

        expect:
        ex.response
        ex.status == HttpStatus.NOT_FOUND
    }
}
```
