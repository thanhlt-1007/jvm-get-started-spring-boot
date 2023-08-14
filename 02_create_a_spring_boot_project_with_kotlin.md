# Create a Spring Boot project with Kotlin

The first part of the tutorial shows you how to create a Spring Boot project in Intellij IDEA using Project Wizard

## Before you start

Download and install the latest version of IntelliJ IDEA Ultimate Edition

If you use IntelliL IDEA Community Edition or another IDE, you can generate a Spring Boot project using a [web-based project generator](https://start.spring.io/)

## Create a Spring Boot project

Create a new Spring Boot Project with Kotlin by using the Project Wizard in IntelliJ IDEA Ultimate Edition:

1. In IntelliJ IDEA, select File | New | Project.

2. In the panel on the left, select New Project | Spring Initializer.

3. Specify the following fields and options in the Project Wizard window:

- Name: demo

- Language: Kotlin

- Build system: Gradle

- JDK: Java 17 JDK

- Java: 17

![create-spring-boot-project.png](https://kotlinlang.org/docs/images/create-spring-boot-project.png)

4. Ensure that you have specified all the fields and click Next.

5. Select the following dependencies that will be required for the tutorial:

- Web / Spring Web

- SQL / Spring Data JDBC

- SQK / H2 Database

![set-up-spring-boot-project.png](https://kotlinlang.org/docs/images/set-up-spring-boot-project.png)

6. Click Create to generate and set up the project.

The IDE will generate and open a new project. It may take some time to download and import the project dependencies.

7. After this, you can observe the following structure in the Project view:

![spring-boot-project-view.png](https://kotlinlang.org/docs/images/spring-boot-project-view.png)

The generated Gradle project corresponds to the Maven's standard directory layout:

- There are packages and classes under the `main/kotlin` folder that belongs to the application.

- The entry point to the application is the `main()` method of the `DemoApplication.kt` file.

## Explore the project Gradle build file

Open the `build.gradle.kts` file: it is the Gradle Kotlin build script, which contains a list of the dependencies required for the application.

The Gradle file is standard for Spring Boot, ut it also contains necessary Kotlin dependencies, including the kotlin-spring Gradle plugin - kotlin`("plugin.spring")`.

Here is the full script with the explanation of all parts and dependencies:

```KOTLIN
// For `KotlinCompile` task below
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
  id("org.springframework.boot") version "3.1.2"
  id("io.spring.dependency-management") version "1.1.2"
  kotlin("jvm") version "1.9.0"                                       // The version of Kotlin to use
  kotlin("plugin.spring") version "1.9.0"                             // The Kotlin Spring plugin
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
  sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
  mavenCentral()
}

dependencies {
  implementation("org.springframework.boot:spring-boot-starter-data-jdbc")
  implementation("org.springframework.boot:spring-boot-starter-web")
  implementation("com.fasterxml.jackson.module:jackson-module-kotli") // Jackson extensions for Kotlin working with JSON
  implementation("org.jetbrains.kotli:kotli-reflect")                 // Kotlin reflection library, required for working with Spring
  runtimeOnly("com.h2database:h2")
  testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<KotlinCompile> {                                       // Settings for `KotlinCompile` tasks
  kotlinOptions {                                                     // Kotlin compiler options
    freeCompileArgs = listOf("-Xjsr305=strict")                       // `-Xjsr305=strict` enables the strict mode for JSR-305 annotations
    jvmTarget = "17"                                                  // This options specifies the target version of the generated JVM bytecode
  }
}

tasks.withType<Test> {
  useJUnitPlatform()
}
```

As you can see, there are a few Kotlin-related artifacts added to the Gradle build file:

1. In the `plugin` block, there are two Kotlin artifacts:

  - `kotlin("jvm")` - the plugin defines the version of Kotlin to be used in the project.

  - `kotlin("plugin.spring")` - Kotlin Spring compiler plugin for adding the `open` modifier to Kotlin classes in order to make them compatible with Spring Frameword features.

2. In the `dependencies` block, a few Kotlin-related modules listed:

  - `com.fasterxml.jackson.module:jackson-module-kotlin` - the module adds support for serialization add deserialization of Kotlin classes and data classes.

  - `org.jetbrains.kotlin:kotlin-reflact` - Kotlin reflaction library.

3.  After the dependencies section, you can see the `KotlinCompile` task configuration block. This is where you can add extra arguments to the compiler to enable or disable various language features.

## Explore the generated Spring Boot application

Open the DemoApplication.kt file:

```KOTLIN
package com.example.demo

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
  runApplication<DemoApplication>(*args)
}
```

### Declaring classes - class DemoApplication

Right after package declaration and import statements you can see the first class declaration, `class DemoApplication`.

In Kotlin, if a class doesn't include any members (properties or functions), you can omit the class body (`{}`) for food.

### @SpringBootApplication annotation

@SpringBootApplication annotation is a convenience annotation in a Spring Boot application. It enables Spring Boot's auto-configuration, component scan, and be able to define an extra configuration on their "application class".

### Program entry point - main()

The `main()` function is the entry point to the application.

It is declared as a top-level function outside the DemoApplication class. The `main()` function invokes the Spring's `runApplication(*args)` function to start the application with the Spring Framework.

### Variable arguments = args: Array<String>

If you check the declaration of the `runApplication()` function, you will see that the parameter of the function is marked with vararg modifier: vararg args: String. This means that you can pass a variable number of String atguments to the function.

### The spread operator - (*args)

The args is a parameter to the `main()` function declared as an array of Strings. Since there is an array of strings, and you want to pass its content to the function, use the spread operator (prefix the array with a start sign *)

## Create a controller

The application is ready to run, but, let's update its logic first.

In the Spring application, a controller is used to handle the web requests. In the `DemoApplication.kt` file, create the `MessageController` class as follows:

```KOTLIN
@RestController
class MessageController {
  @GetMapping("/")
  fun index(@RequestParam("name") name: String) = "Hello, $name!"
}
```

### @RestController annotation

You need to tell Spring that `MessageController` is a REST Controller, so you should mark it with the @RestController annotation.

This annotation means this class will be picked up by the component scan because it's on the same package as our DemoApplication class.

### @GetMapping annotation

`@GetMapping` marks the functions of the REST controller that implement the endpoints corresponding to HTTP GET calls:

```KOTLIN
@GetMapping("/")
fun index(@RequestParam("name") name: String) = "Hello, $name!"
```

### @RequestParam annotation

The function parameter `name` is marked with `@RequestParam` annotation. This annotation indicates that a method parameter should be bound to a web request parameter.

Hence, if you access the application at the toor and supply a request parameter called "name", like "/?name=<your-name>", the parameter value will be used as an argument for invoking the `index()` function.

### Single-expression functions - index()

Since `index()` function contains only one statement you can declare it as a single-expression function.

This mean the curly braces can be ommited and the body is specified after the equals sign `=`.

### Type interface for function return types

The `index()` functions does not declare the return type explicitly. Instead, the compiler infers the return type by looking at the result of the statement on the right hand side from the equals sign `=`.

The type of `Hello, $name!` expression is `String`, hence the return type of the function is also `String`.

### String template - $name

`Hello, $name!` expression is called a String template in Kotlin.

String templates are String literal that contain embedded expressions.

This is a convenient replacement for String concatenation operations.

These Spring annotations also require addtional imports:

```KOTLIN
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.bind.annotation.RestController
```

Here is a complete code of `DemoApplication.kt`:

```KOTLIN
package com.example.demo

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@SpringBootApplication
class DemoApplication

fun main(args: Array<String>) {
  runApplication<DemoApplication>(*args)
}

@RestController
class MessageController {
  @GetMapping("/")
  fun index(@RequestParam("name") name: String) = "Hello, $name!"
}
```

## Run the application

The Spring application is now ready to run:

1. Click the gren Run icon in the gutter beside the `main()` method:

![run-spring-boot-application.png](https://kotlinlang.org/docs/images/run-spring-boot-application.png)

You can also run the

```BASH
./gradlew bootRun
```

command in the terminal.

This starts the local server on your computer.

2. Once the application starts, open the following URL:

```BASH
http://localhost:8080?name=John
```

You shoud see "Hello, John" printed as a response:

![spring-application-response.png](https://kotlinlang.org/docs/images/spring-application-response.png)
