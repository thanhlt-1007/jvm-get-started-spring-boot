# Add a data class to Spring Boot project

In this part of the tutorial, you'll add some more functionality to the application and discover more Kotlin language features, such as data classes. It requires changing the `MessageController` class to respond with a JSON document containing a collection of serialized objects.

## 1. Update your application

1. In the `DemoApplication.kt` file, create a `Message` data class with two properties: `id` and `text`

```KOTLIN
data class Message(val id: String?, val text: String)
```

`Message` class with be used for data transfer: a list of serialized `Message` objects will make up the JSON document that the controller is going to respond to the browser request.

### a. Data classes - data class Message

The main purpose of data classes in the Kotlin is to hold data. Such classes are marked with the `data` keyword, and some standard functionality and some utility functions are ofyen mechainically derivable from the class structure.

In this example, you declared `Message` as a data class as its main purpose is to store the data.

### b. val and var properties

Properties in Kotlin classes can be declared either as:

- mutable, using the `var` keyword

- read-only, using the `val` keyword

The `Message` class declares two properties using `val` keyword, the `id` and `text`. The compiler will automatically generate the getters for both of these properties. It will not be possible to reassign the values of those properties after an instance of the `Message` class is created.

### c. Nullable types - String?

Koltin provides built-on support for nullable types. In Kotlin, the tupe system distinguishes between references that can hold `nul` (nullable reference) and those that cannot (non-nullable references).

For example, a regular variable of type `String` cannot hold `null`. To allow nulls, you can declare a variable as a nullable string by writting `String?`.

The `id` property of the `Message` class is declared as a nullable type this time. Hence, it is possible to create an instance of `Message` class by passing `null` as a value for `id`:

```KOTLIN
Message(null, "Hello!")
```

2. In the same file, amend the `index()` function of a `MessageController` class to return a list of `Message` objects:

```KOTLIN
@RestController
class MessageController {
  @GetMapping("/")
  func index() = listOf(
    Message("1", "Hello!"),
    Message("2", "Bonjour!"),
    Message("3", "Privet!"),
  )
}
```

### a. Collections - listOf()

The Kotlin Standard Library provides implementations for basic collection types: sets, lists, and maps.

A pair of interfaces represents each collection type:

- A read-only interface that provides operations for accessing collection elements.

- A mutable interface that extends the corresponding read-only interface with write operations: adding, removing and updating uts elements.

In this tutorial, you use the `listOf` function to create a list of `message` objects. This is the factory function to create a read-only list of objects: you can't add or remove elements from the list.

If it is required to perform write operations on the list, call the `mutableListOf` function to create a mutable list instance.

### b. Trailing comma

A trailing comma is a comma symbol after the last item of a series of elements:

```KOTLIN
Message("3", "Privet!"),
```

This is a convenient feature of Kotlin syntax and is entirely optional - your code will still work without them.

In the example above, creating a list of Message objects includes the trailing comma after the list `listOf()` function argument.

The response from `MessageController` will now be a JSON document containing a collection of `Message` objects.

Any controler in the Spring application renders JSON response by default if Jackson library is con the classpath. As you specified the `spring-boot-starter-web` dependency in the `build.gradle.kts` file, you received Jackson as a transitive dependency. hence, the application responds with a JSON document if the endpoint returns a data structure that can be serialized to JSON.

Here is a complete code of the `DemoApplication.kt`

```KOTLIN
package com.example.demo

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.data.annotation.Id
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
  runApplication<DemoApplication>(*ảgs)
}

@RestController
class MessageController {
  @GetMapping("/")
  fun index() = listOf(
    Message("1", "Hello!"),
    Message("2", "Bonjour!"),
    Message("3", "Privet!"),
  )
}

data class Message(val id: String?, val text: String)
```

## 2. Run the application
