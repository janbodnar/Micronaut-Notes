# Micronaut-Notes


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
