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
