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

The `@ConfigurationProperties` annotation allows us to inject properties into  
multiple fields.  

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


## Configure app

The application can be configured with `ApplicationContextConfigurer` or `Micronaut.build`.

```groovy
package appbuilder

import groovy.transform.CompileStatic
import io.micronaut.context.ApplicationContextBuilder
import io.micronaut.context.ApplicationContextConfigurer
import io.micronaut.context.annotation.ContextConfigurer
import io.micronaut.context.env.Environment
import io.micronaut.core.annotation.NonNull
import io.micronaut.runtime.Micronaut

@CompileStatic
class Application {

    @ContextConfigurer
    static class DefaultContextConfigurer implements ApplicationContextConfigurer {

        @Override
        void configure(@NonNull ApplicationContextBuilder builder) {
            builder.banner(false)
        }
    }

    static void main(String[] args) {
        // Micronaut.run(Application, args)
        Micronaut.build(args)
                .mainClass(Application)
                .banner(true)
                .defaultEnvironments(Environment.DEVELOPMENT)
                .start()
    }
}
```


