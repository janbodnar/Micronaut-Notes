# Filters 

## Test filter order

```java
package com.baeldung.micronaut.httpfilters;

import static org.junit.jupiter.api.Assertions.assertDoesNotThrow;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.stream.Collectors;

import org.junit.jupiter.api.Test;

import com.baeldung.micronaut.httpfilters.bean.MemoryLogger;

import ch.qos.logback.classic.spi.ILoggingEvent;
import io.micronaut.http.HttpRequest;
import io.micronaut.http.client.BlockingHttpClient;
import io.micronaut.http.client.HttpClient;
import io.micronaut.http.client.annotation.Client;
import io.micronaut.runtime.EmbeddedApplication;
import io.micronaut.test.extensions.junit5.annotation.MicronautTest;
import jakarta.inject.Inject;

@MicronautTest
class OrderFilterTest {

    @Inject
    EmbeddedApplication<?> application;

    @Inject
    MemoryLogger mlog;

    @Test
    void testOrderFilters(@Client("/") HttpClient httpClient) {

        BlockingHttpClient client = httpClient.toBlocking();
        assertDoesNotThrow(() -> client.retrieve(HttpRequest.GET("/order-filter")));
        String filterOrder = mlog.getAppender().getEvents().stream()
                .map(ILoggingEvent::getFormattedMessage)
                .collect(Collectors.joining(" "));

        assertTrue("OrderFilter2 OrderFilter1".equals(filterOrder));

        mlog.stopAppender();
    }
}
```
