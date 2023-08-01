# Micronaut annotations

## @Body

The `@Body` annotation loads the request body into the given parameter.  

```groovy
package bodyan.controller

import io.micronaut.http.HttpStatus
import io.micronaut.http.MediaType
import io.micronaut.http.annotation.Body
import io.micronaut.http.annotation.Controller
import io.micronaut.http.annotation.Get
import io.micronaut.http.annotation.Post
import io.micronaut.http.annotation.Status

@Controller
class MyController {

    @Get(uri="/", produces="text/plain")
    String index() {
        "Home page"
    }

    @Post(value = "/echo", consumes = MediaType.TEXT_PLAIN, produces = MediaType.TEXT_PLAIN)
    @Status(HttpStatus.OK)
    String hello(@Body String text) {
        return text
//        HttpResponse.ok(text).header(CONTENT_TYPE, MediaType.TEXT_PLAIN)
    }

}
```

The `@Body` annotation in the `hello` method takes the request body and inserts it into the `text` parameter.  

```groovy
package controller

import io.micronaut.http.HttpRequest
import io.micronaut.http.HttpResponse
import io.micronaut.http.HttpStatus
import io.micronaut.http.MediaType
import io.micronaut.http.client.HttpClient
import io.micronaut.http.client.annotation.Client
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

    @Shared @AutoCleanup @Inject @Client("/")
    HttpClient client

    void "test index"() {
        given:
        String payload = 'sample message'
        HttpResponse res = client.toBlocking()
                .exchange(HttpRequest.POST('/echo', payload).contentType(MediaType.TEXT_PLAIN), String)

        expect:
        res.status == HttpStatus.OK
        res.getBody(String).get() == payload
    }
}
```

This curl command makes a POST request with content type `text/plain`.  
`$ curl -X POST -H "Content-Type: text/plain" --data "hello there" localhost:61748/echo`  






