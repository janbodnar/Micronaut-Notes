# Configuration

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
