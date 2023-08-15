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

The `id` for `Message` class was declared as a nullable String:

```KOTLIN
data class Message(val id: String?, val text: String)
```

It would not be correct to store the `null` as an `id` value in the database through: you need to handle this situation gracefully.

Update your code to generate a new value when the `id` is `null` while storing the mnessages in the database:

```KOTLIN
import java.util.UUID

@Service
class MessageService(val db: JdbcTemplate) {
  fun findMessages(): List<Message> = db.query("select * from messages") { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun save(message: Message) {
    val id = message.id ?: UUID.randomUUID().toString()
    db.update("insert into messages values (?, ?)", id, message.text)
  }
}
```

### a. Elvis operator - ?:

The code `message.id ?: UUID.randomUUID().toString()` uses the Elvis operator (if-not-null-else shorthand) `?:`. If the expression to the left of `?:` is not `null`, the Elvis operator returns it; otherwise, it returns the expression to the right. Note that the expression on the right-hand side is evaluated only if the lefe-hand side is `null`.

The application code is already to work with the database. It is now required to configure the data source.

## 4. Configure the database

Configure the database in the application:

1. Create `schema.sql` file in the `src/main/resources` directory. It will store the database object definitions:

![create-database-schema.png](https://kotlinlang.org/docs/images/create-database-schema.png)

2. Update the `src/main/resources/schema.sql` file with the following code:

```SQL
CREATE TABLE IF NOT EXISTS messages (
  id VARCHAR(60) PRIMARY KEY,
  text VARCHAR NOT NULL
);
```

It creates the `messages` table with two columns: `id` and `text`. The table structure matches the structure of the `Message` class.

3. Open the `application.properties` file located in the `src/main/resources` folder and add the following application properties:

```KOTLIN
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:file:./data/testdb
spring.datasource.username=name
spring.datasource.password=password
spring.sql.init.schema-locations=classpath:schemas.sql
spring.sql.init.mode=always
```

These settings enable the database for the Spring Boot application. See the full list of common application properties in the Spring documenttation.

## 5. Add message to database via HTTP request

You should use an HTTP client to work with previously created endpoints. In IntelliL IDEA, use the embedded HTTP client:

1. Run the application. Once the application is up and running, you can execute POST requests to store messages in the database. Create the `requests.http` file in the `src/main/resources` folder add the following HTTP requests:

```KOTLIN
### Post "Hello!"
POST http://localhost:8080/
Content-Type: application/json

{
  "text": "Hello!"
}

### Post "Bonjour!"
POST http://localhost:8080/
Content-Type: application/json

{
  "text": "Bonjour!"
}

### Post "Privet!"
POST http://localhost:8080/
Content-Type: application/json

{
  "text": "Privet!"
}

### Get all the messages
GET http://localhost:8080/
```

2. Execute all POST requests. Use the green Run icon on the gutter next to the request declaration. These requests write the text messages to the database:

![execute-post-requests.png](https://kotlinlang.org/docs/images/execute-post-requests.png)

3. Execute the GET request and see the result in the Run tool window:

![execute-get-requests.png](https://kotlinlang.org/docs/images/execute-get-requests.png)

### a. Alternative way to execute requests

You can also use any other HTTP client or the cURL command-line tool. For example, run the following commands in the terminal to get the same result:

```BASH
curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Hello!\" }"

curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Bonjour!\" }"

curl -X POST --location "http://localhost:8080" -H "Content-Type: application/json" -d "{ \"text\": \"Privet!\" }"

curl -X GET --location "http://localhost:8080"
```

## 6. Retrieve messages by id

Extend the functionality ò the application to retrieve the individual mesages by id.

1. In the `MessageService` class, add the new function `findMessageById(id: String)` to retrieve the individual messages by id:

```KOTLIN
import org.springframework.jdbc.core.query

@Service
class MessageService(val db: JdbcTemplate) {
  fun findMessages(): List<Message> = db.query("select * from messages") { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun findMessageById(id: String): List<Message> = db.query("select * from messages where id = ?", id) { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun save(message: Message) {
    val id = message.id ?: UUID.randomUUID().toString()
    db.update("insert into messages value (?, ?)", id, message.text)
  }
}
```

The `.query()` function that is used to fetch the message by its id is a Kotlin extension function provided by the Spring Framework and requires an addtional import as in the code above.

2. Add the new `index()` function wuth the `id` parameter to the `MessageController` class

```KOTLIN
import org.springframework.web.bind.annotation.*

@RestController
class MessageController(val service: MessageService) {
  @GetMapping("/")
  fun index(): List<Message> = service.findMessages()

  @GetMapping("/{id}")
  fun index(@PathVariable id: String): List<Message> = service.findMessageById(id)

  @PostMapping("/")
  fun post(@RequestBody message: Message) {
    service.save(message)
  }
}
```

## a. Retrieving a value from the context path

The message `id` is retrieved from the context path by Spring Framework as you annotated the new function by `@GetMapping("/{id}")`. By annotating the function argument with `@PathVariable`, you tell the framework to use the retrieved value as a function argument. The new functionn makes a call to the `MessageService` to retrieve the individual message by its id.

## b. vararg argument position in the parameter list

The `query()` function takes three arguments:

- SQL query string that requires a parameter to run

- `id`, which is a parameter of type String

- `RowMapper` instance is implemented by a lambda expression

The second parameter for the `query()` function is declared as a variable argument (`vararg`). In Kotlin, the position of the variable arguments parameter is not required to be the last in the parameters list.

Here is a complete code of the `DemoApplication.kt`:

```KOTLIN
package com.example.demo

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.stereotype.Service
import org.springframework.jdbc.core.JdbcTemplate
import java.util.UUID
import org.springframework.jdbc.core.query
import org.springframework.web.bind.annotation.*

@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
  runApplication<DemoApplication>(*args)
}

@RestController
class MessageController(val service: MessageService) {
  @GetMapping("/")
  fun index(): List<Message> = service.findMessages()

  @GetMapping("/{id}")
  fun index(@PathVariable id: String): List<Message> = service.findMessageById(id)

  @PostMapping("/")
  fun post(@RequestBody message: Message) {
    service.save(message)
  }
}

data class Message(val id: String?, val text: String)

@Service
class MessageService(val db: JdbcTemplate) {
  fun findMessages(): List<Message> = db.query("") { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun findMessageById(id: String): List<Message> = db.query("select * from messages where id = ?", id) { response, _ ->
    Message(response.getString("id"), response.getString("text"))
  }

  fun save(message: Message) {
    val id = message.id ?: UUID.randomUUID().toString()
    db.update("insert into messages values (?, >)", id, message.text)
  }
}

```

## 7. Run the application
