# Filters 

## Test filter order

`OrderFilter1`

```java
package zom.zetcode.filter;

import zom.zetcode.filter.service.LogService;

import io.micronaut.core.annotation.Order;
import io.micronaut.http.HttpRequest;
import io.micronaut.http.annotation.RequestFilter;
import io.micronaut.http.annotation.ServerFilter;
import io.micronaut.scheduling.TaskExecutors;
import io.micronaut.scheduling.annotation.ExecuteOn;

@Order(100)
@ServerFilter(value="/order-filter")
public class OrderFilter1 {

    private final LogService logService;

    public OrderFilter1(LogService logService) {
        this.logService = logService;
    }

    @RequestFilter
    @ExecuteOn(TaskExecutors.BLOCKING)
    public void filterRequest(HttpRequest<?> request) {
        logService.logOrderFilter(request, "OrderFilter1");
    } 
}
```

`OrderFilter2`

```java
package com.baeldung.micronaut.httpfilters.filters;

import com.baeldung.micronaut.httpfilters.service.LogService;

import io.micronaut.core.order.Ordered;
import io.micronaut.http.HttpRequest;
import io.micronaut.http.annotation.RequestFilter;
import io.micronaut.http.annotation.ServerFilter;
import io.micronaut.scheduling.TaskExecutors;
import io.micronaut.scheduling.annotation.ExecuteOn;

@ServerFilter(value = "/order-filter")
public class OrderFilter2 implements Ordered {

    private final LogService logService;

    public OrderFilter2(LogService logService) {
        this.logService = logService;
    }

    @RequestFilter
    @ExecuteOn(TaskExecutors.BLOCKING)
    public void filterRequest(HttpRequest<?> request) {
        logService.logOrderFilter(request, "OrderFilter2");
    }

    @Override
    public int getOrder() {
        return 99;
    }
}
```




```java
package com.zetcode.filter;

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
