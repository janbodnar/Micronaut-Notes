# Micronaut-Notes





changing port in `application.properties`: `micronaut.server.port=8081`  
setting development environment - `MICRONAUT_ENVIRONMENTS=dev ./gradlew run`  


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








