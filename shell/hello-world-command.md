# Hello World Command

We are going to create a simple command that prints `Hello World` on the command line using Spring Shell.

Lets create `build.gradle` fill with all required dependencies.

```
group 'shell'
version '0.0.1'

buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE'
    }
}

apply plugin: 'java'
apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
    maven {
        url 'https://repo.spring.io/libs-milestone'
    }
}

dependencies {
    compile 'org.springframework.shell:spring-shell-starter:2.0.0.M2'
    compile "org.springframework:spring-web:4.3.13.RELEASE"
}
```

Then we need to create the application class, that will start up the shell. 

```
package shell;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

Because we are using Spring Boot, the application will now get all the beans that are annotated with `@ShellComponent` annotation and will try to use them as commands in the shell. Lets create one. 

```
package shell.command;

import org.springframework.shell.standard.ShellComponent;
import org.springframework.shell.standard.ShellMethod;
import org.springframework.shell.standard.ShellOption;

@ShellComponent
public class TestCommand {

    @ShellMethod(key = "test", value = "Test command")
    public String test() {
        return "Hello world!";
    }
}

```

 Now we can build the application and test the command.

```
$ gradle clean build
$ java -jar build/libs/shell-0.0.1.jar
```



