# Micronaut-Notes

## CLI

`mn create-app --list-features` - list all features  

## Return text 

We use the `@Produces` annotation with `MediatType.TEXT_PLAIN`. 

```groovy
@Get('hello')
@Produces(MediaType.TEXT_PLAIN)
String index() {
    return 'hello there!'
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
## Create bean via factory

```groovy
package bean

class Hello implements IHello {

    @Override
    String sayHello() {
        return "hello there!"
    }
}

---

package bean

interface IHello {

    String sayHello()
}

---

package factory

import bean.Hello
import bean.IHello
import io.micronaut.context.annotation.Factory
import jakarta.inject.Singleton

@Factory
class MyBeanFactory {
    @Singleton
    IHello getHelloBean() {
        return new Hello()
    }
}
```








