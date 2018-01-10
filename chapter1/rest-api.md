# REST API

We are going to create a simple REST API web application using Spring 5.

Lets first create build.gradle file in the project root directory.

```
buildscript {
  ext {
    springBootVersion = '2.0.0.M7'
  }
  repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/snapshot" }
    maven { url "https://repo.spring.io/milestone" }
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
  maven { url "https://repo.spring.io/snapshot" }
  maven { url "https://repo.spring.io/milestone" }
}


dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

Now lets create the application that will be used to start up our server. 

```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

The last step is to create REST API controller that will respond to user requests. 

```
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

   @GetMapping("/hello")
   public String sayHello() {
      return "Hello";
   }
}
```

Now we can build the code. Run this command in the project root, where the `build.gradle` file is located.

```
➜ gradle clean build
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :test
2018-01-10 09:14:09.499  INFO 78584 --- [       Thread-5] o.s.w.c.s.GenericWebApplicationContext   : Closing org.springframework.web.context.support.GenericWebApplicationContext@7baa7756: startup date [Wed Jan 10 09:14:07 PST 2018]; root of context hierarchy


BUILD SUCCESSFUL in 16s
6 actionable tasks: 5 executed, 1 up-to-date
```

Then we have couple of ways to run the application. If we want to run the application for development, we can use Gradle task `bootRun`.

```
gradle bootRun
```

> If we want to see all the tasks available, run `gradle tasks --all`

If we want to run the application without Gradle, we can use `java -jar` from the command line.

```
java -jar build/libs/demo-0.0.1-SNAPSHOT.jar
```

After the application is started up, we can try to call the api from the command line. 

```
➜ curl localhost:8080/hello
Hello%
```



