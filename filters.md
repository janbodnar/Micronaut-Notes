# Filters 

## Test filter order

`MemoryAppender`

```java
package com.zetcode.bean;

import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.AppenderBase;

import java.util.ArrayList;
import java.util.List;

public class MemoryAppender extends AppenderBase<ILoggingEvent> {

    private final List<ILoggingEvent> events = new ArrayList<>();

    @Override
    protected void append(ILoggingEvent e) {
        events.add(e);
    }

    public List<ILoggingEvent> getEvents() {
        return events;
    }
}
```


`MemoryLogger`

```java
package com.zetcode.bean;

import org.slf4j.LoggerFactory;

import com.zetcode.service.LogService;

import ch.qos.logback.classic.Logger;
import io.micronaut.context.annotation.Prototype;

@Prototype
public class MemoryLogger {

    private final Logger log;
    private final MemoryAppender appender;

    public MemoryLogger() {

        log = (Logger) LoggerFactory.getLogger(LogService.class);
        appender = new MemoryAppender();
        log.addAppender(appender);
        appender.start();
    }

    public MemoryAppender getAppender() {
        return appender;
    }

    public void stopAppender() {
        appender.stop();
    }
}
```

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
package com.zetcode.filter;

import com.zetcode.service.LogService;

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


`OrderFilterTest`

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
