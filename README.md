# Micronaut-Notes

changing port in `application.properties`: `micronaut.server.port=8081`  

## MIcronaut version

Using `VersionUtils`.

```java
@Get("/micronaut-version")
public Map<String, String> micronautVersion() {

    String micronautVersion = VersionUtils.MICRONAUT_VERSION;
    return Map.of("micronautVersion", micronautVersion);
}
```


## Set env with app builder

```java
package example;

import io.micronaut.runtime.Micronaut;

public class Application {

    public static void main(String[] args) {
        Micronaut.build(args)
                .defaultEnvironments("dev")
                .mainClass(Application.class)
                .start();
    }
}
```

## Favicon

place `favicon.ico` into `src/main/resources/static` and configure the router:

```yaml
micronaut:
  router:
    static-resources:
      default:
        enabled: true
        mapping: "/**"
        paths: "classpath:static"
```

## Start H2 console 

```java
package example.web;

import org.h2.tools.Server;
import io.micronaut.context.annotation.Value;

import jakarta.annotation.PostConstruct;
import io.micronaut.context.annotation.Context;
import io.micronaut.context.annotation.Requires;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Context
@Requires(env = "dev")
public class H2ConsoleStarter {

    private static final Logger LOG = LoggerFactory.getLogger(H2ConsoleStarter.class);

    private final int port;

    public H2ConsoleStarter(@Value("${h2.console.port:8082}") int port) {
        this.port = port;
    }

    @PostConstruct
    public void startConsole() {
        LOG.info("Starting H2 console at http://localhost:{}", port);
        try {
            Server.createWebServer("-web", "-webAllowOthers", "-webPort", String.valueOf(port)).start();
            LOG.info("H2 console started");
        } catch (Exception e) {
            LOG.error("Failed to start H2 console on port {}", port, e);
        }
    }
}
```

## Get property

```groovy
package getprop

import io.micronaut.runtime.Micronaut
import io.micronaut.context.ApplicationContext
import groovy.transform.CompileStatic

@CompileStatic
class Application {

    static void main(String[] args) {
        ApplicationContext ctx =  Micronaut.run(Application, args)
        ctx.getProperty('micronaut.application.name', String).ifPresent {println(it)}
    }
}
```

## Environment 

The current application environment. The environment represents the loaded configuration  
of the application for a current list of active environment names.  

 A *default environment* is one that is only applied if no other environments are explicitly  
 specified or deduced.

Setting environments via System property:  
`$ java -Dmicronaut.environments=dev,qa -jar build\libs\environment-0.1-all.jar`  

```groovy
package com.zetcode.start

import io.micronaut.context.annotation.Property
import io.micronaut.context.env.Environment
import io.micronaut.context.event.StartupEvent
import io.micronaut.runtime.event.annotation.EventListener
import jakarta.inject.Inject
import jakarta.inject.Singleton

@Singleton
class AppStartListener {

    @Inject
    private Environment env

    @EventListener
    void onStartup(StartupEvent event) {
        println env.activeNames
        println env.getActiveNames()
    }
}
```

Properties can be retrieved from the `Environment` through `get` or `getProperty`.  

```groovy
@Inject
private Environment env

@EventListener
void onStartup(StartupEvent event) {

    println env.get('app.message', String).orElse('message not set')
    println env.getProperty('app.message', String).orElse('message not set')
}
```


## Processing an exception in test 

Endpoint  

```groovy
@Controller("/greeting")
public class GreetingController {

    @Get("/error")
    public HttpResponse<String> errorEndpoint() {
        return HttpResponse.status(HttpStatus.UNPROCESSABLE_ENTITY).body("ERROR");
    }

}
```

Test  

```groovy
def "enpoint should throw HttpClientResponseException"() {
    when:
    HttpResponse response = client.toBlocking().exchange("/greeting/error", String)

    then:
    HttpClientResponseException ex = thrown()
    and:
    ex.status == HttpStatus.UNPROCESSABLE_ENTITY
    and:
    ex.response.body == Optional.of('ERROR')
}
```

## Enable gradle terminal test output

```gradle
tasks.withType(Test).configureEach {
    testLogging {
        // show basic events and standard streams so logs from tests are visible
        events "passed", "skipped", "failed"
        showStandardStreams = true
    }
}
```


## H2 database 

Initialize H2 database in `application.properties`  

Either separate into schema and data  
`datasources.default.url=jdbc:h2:mem:testdb;INIT=RUNSCRIPT FROM 'classpath:/schema.sql'\\;RUNSCRIPT FROM 'classpath:/data.sql'`   

or in one SQL script  
`datasources.default.url=jdbc:h2:mem:testdb;INIT=RUNSCRIPT FROM 'classpath:/init_db.sql'`  

Default schema generation must be turned off  
`#datasources.default.schema-generate=CREATE_DROP`

The table name and the table column names must match the naming strategy of the entities.  

```groovy
package repojdbc.model

import io.micronaut.core.annotation.Nullable
import io.micronaut.data.annotation.GeneratedValue
import io.micronaut.data.annotation.Id
import io.micronaut.data.annotation.MappedEntity
import io.micronaut.data.model.naming.NamingStrategies

@MappedEntity(value = 'users', namingStrategy = NamingStrategies.LowerCase)
class User {

    @Id
    @Nullable
    @GeneratedValue
    Long id

    String firstName
    String lastName
    String occupation

    User(String firstName, String lastName, String occupation) {
        this.firstName = firstName
        this.lastName = lastName
        this.occupation = occupation
    }
}
```

## Event listenter 

Execute code at application startup  

```groovy
package eventlist.listener

import io.micronaut.context.event.StartupEvent
import io.micronaut.runtime.event.annotation.EventListener
import jakarta.inject.Singleton

@Singleton
class StartUpListener {

    @EventListener
    final void onStartup(StartupEvent event) {
        println 'application started'
    }
}
```

## Test repository

Repository with a query expression  

```groovy
package repojdbc.resository

import io.micronaut.data.jdbc.annotation.JdbcRepository
import io.micronaut.data.model.query.builder.sql.Dialect
import io.micronaut.data.repository.CrudRepository
import repojdbc.model.User

@JdbcRepository(dialect=Dialect.H2)
interface UserRepository extends CrudRepository<User, Long> {

    List<User> findByLastNameStartsWith(String str)
}
```

Testing class  

```groovy
package repojdbc

import io.micronaut.test.extensions.spock.annotation.MicronautTest
import jakarta.inject.Inject
import repojdbc.resository.UserRepository
import spock.lang.Specification

@MicronautTest
class RepojdbcSpec extends Specification {

    @Inject
    UserRepository repository

    def 'find users by last name starting with D'() {
        given:
        def data = repository.findByLastNameStartsWith('D')

        expect:
        data.size() == 4
        data.get(0).lastName == 'Doe'
    }
}
```

The `init_db.sql` file  

```sql
CREATE TABLE users(id BIGINT PRIMARY KEY AUTO_INCREMENT,
    firstName VARCHAR(255), lastName VARCHAR(255), occupation VARCHAR(255) );

INSERT INTO users(firstname, lastname, occupation) VALUES('John', 'Doe', 'gardener');
INSERT INTO users(firstname, lastname, occupation) VALUES('Jane', 'Doe', 'teacher');
INSERT INTO users(firstname, lastname, occupation) VALUES('Peter', 'Maly', 'shopkeeper');
INSERT INTO users(firstname, lastname, occupation) VALUES('Roger', 'Roe', 'driver');
```


## Control panel 

    developmentOnly("io.micronaut.controlpanel:micronaut-control-panel-ui")
    developmentOnly("io.micronaut.controlpanel:micronaut-control-panel-management")

    Micronaut.build(args).mainClass(Application).defaultEnvironments('dev').start()

`localhost:8080/control-panel`
   
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

## Pebble view

The controller sets the name of the view with `@View`. The default extension for  
Pebble views is html. 

```groovy
package pebblelist.controller

import io.micronaut.http.HttpResponse
import io.micronaut.http.annotation.Controller
import io.micronaut.http.annotation.Get
import io.micronaut.views.View

@Controller
class MyController {

    @View("users")
    @Get("/users")
    HttpResponse<?> index() {
        def data = ['users' : [new User("John Doe", 'gardener'),
                    new User('Roger Roe', 'driver')]]
        return HttpResponse.ok(data)
    }
}

record User(String name, String occupation) {}
```

The views are located in `src/resources/views` by default

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Users</title>
</head>
<body>

    <table>
        <thead>
            <tr>
            <th>name</th>
            <th>occupation</th>
            </tr>
        </thead>
        <tbody>
        {% for u in users %}
        <tr>
            <td>{{u.name}}</td>
            <td>{{u.occupation}}</td>
        </tr>
        {% endfor %}

        </tbody>

    </table>

</body>
</html>
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








