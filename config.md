# Configuration

## Property injection

Properties can be injected with `@Value` or `@Property`.  
The `@Value` annotation *requires* single quotes.  
We can add default values if properties are missing.  

```groovy
import io.micronaut.runtime.event.annotation.EventListener
import jakarta.inject.Singleton

@Singleton
class StartUpListener {

    @Value('${zetcode.name:Unknown}')
    String name

    @Property(name='zetcode.msg')
    String msg

    @Property(name='zetcode.colour', defaultValue = 'blue')
    String colour

    @EventListener
    final void onStartup(StartupEvent event) {
        println 'application started'

        println name
        println msg
        println colour
    }
}
```

## @ConfigurationProperties

The configuration class  

```groovy
package com.zetcode.config

import io.micronaut.context.annotation.ConfigurationProperties

@ConfigurationProperties('zetcode')
class MyConfig {
    String name
    String message
    List<String> colours
}
```
The controller  

```groovy
package com.zetcode.controller

import com.zetcode.config.MyConfig
import io.micronaut.http.annotation.Controller
import io.micronaut.http.annotation.Get
import jakarta.inject.Inject

@Controller("/test-config")
class MyController {

    @Inject MyConfig cfg

    @Get(uri="/", produces="text/plain")
    String index() {
        [cfg.name, cfg.message, cfg.colours]
    }
}
```

The configuration data in the `application.yml` file  

```groovy
micronaut:
  application:
    name: config

zetcode:
  name: John Doe
  message: 'hello there!'
  colours: ['red', 'blue', 'white']
```

Test  

```groovy
package config

import com.zetcode.config.MyConfig
import io.micronaut.runtime.EmbeddedApplication
import io.micronaut.test.extensions.spock.annotation.MicronautTest
import spock.lang.Specification
import jakarta.inject.Inject

@MicronautTest
class ConfigSpec extends Specification {

    @Inject
    EmbeddedApplication<?> application

    void "test app configuration"() {
        when:
        def cfg = application.applicationContext.getBean(MyConfig)

        then:
        cfg.name == "John Doe"
        cfg.message == "hello there!"
        cfg.colours == ['red', 'blue', 'white']
        
    }
}
```
