# Add database support for Spring Boot project

In this part of the tutorial, you'll add and configure a database to your project using JDBC. In JVM applications, you use JBDC interact withdatabases. For convenience, the Spring Framework orivudes the `JdbcTemplate` class that simplifies the use of JDBC and helps to avoid common errors.

## 1. Add database support

The common practice in Spring Framwork based applications is to implement the database access logic within the so-called service layer - this is where business logic lives. In Spring, you should mark classes with the `@Service` annotation to imply that the class belongs to the service layer of the application. In this application, you will create the `MessageService` class for this purpose.

In the `DemoApplication.kt` file, create the `MessageService` class as follows:

```KOTLIN
import org.springframework.stereotype.Service
import org.springframework.jdbc.core.JdbcTemplate

@Service
class MessageService(val db: JdbcTemplate) {
  fun findMessages(): List<Message> = db.query("select * from messages") { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun save(message: Message) {
    db.update("insert into messages values (?, ?)", message.id, message.text)
  }
}
```

### a. Constructor argument and dependency injection - (val db: JdbcTemplate)

A class in Kotlin has a primary constructor. It can also have one or more secondary constructors. The primary constructor is a part of the class header, and it goes after the class name and optional type parameters. In our case, the constructor is `(val db: JdbcTemplate)`.

`val db: JdbcTemplate` is the constructor's argument:

```KOTLIN
@Service
class MessageService(val db: JdbcTemplate)
```

### b. Trailing lambda and SAM conversion

The `findMessages()` function calls the `query()` function of the `JdbcTemplate` class. The `query()` function takes two arguments: an SQL query a String instance, and a callback that will map one object per row:

```KOTLIN
db.query("...", RowMapper { ... })
```

The `RowMapper` interface declares only one method, so it is possible to implement it via lamdba expression by omitting the name of the interface. The Kotlin compiler knows the interface that the lambda expression needs to be convert to because you see it as a parameter for the function call. This is known as SAM conversion in Kotlin.

```KOTLIN
db.query("...", { ... })
```

After the SAM conversion, the query function ends up with two arguments: a String at the first position, and a lambda expression at the last position. According to the Kotlin convention, if the last parameter of a function is a function, then a lambda expression passed as the corresponding argument can be replaced outside the parenthese. Such syntax is also known as trailing lambda:

```KOTLIN
db.query("...", { ... })
```

### c. Underscore for unused lambda argument

For a lambda with multiple parameters, you can use the underscore `_` character to replace the names of the parameters you don't use.

Hence, the final syntax for query function call looks like this:

```KOTLIN
db.query("select * from messages") { response, _ -> Message(response.getString("id"), response.getString("text")) }
```

## 2. Update the MessageController class

Update `MessageController` to use the new `MessageService` class:

```KOTLIN
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.PostMapping

@RestController
class MessageController(val service: MessageService) {
  @GetMapping("/")
  fun index(): List<Message> = service.findMessages()

  @PostMapping("/")
  fun post(@PostMapping message: Message) {
    service.save(message)
  }
}
```

### a. @PostMapping annotation

The method responsible for handling HTTP POST requests needs to be annotated with `@PostMapping` annotation. To be able to convert the JSON sent as HTTP Body content into an object, you need to use the `@RequestBody` annotation for the method argument. Thanks to having Jackson library in the classpath of the application, the conversion happens automatically.

## 3. Update the MessageService class

## 4. Configure the database

## 5. Add message to database via HTTP request

### a. Alternative way to execute requests

## 6. Retrieve messages by id

## 7. Run the application
