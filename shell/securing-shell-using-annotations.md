# Securing Shell using Annotations

We are going to have a look at how to create our own annotation to secure some commands. When we annotate a command with our annotation, it will make sure that user needs to authenticated in order to use it. It is going to look like this. 

```
@Secured
@ShellMethod(key = "logout", value = "Logout from GitHub. Usage: logout")
public void logout() {
    sessionHolder.removeCurrentSession();
}
```

First, create `build.gradle` with all needed dependencies.

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
    compile "org.springframework:spring-aop:4.3.13.RELEASE"
    compile "org.springframework:spring-web:4.3.13.RELEASE"
    compile "org.aspectj:aspectjrt:1.8.13"
    compile "org.aspectj:aspectjweaver:1.8.13"
}
```

Now we need to create the new annotation `@Secured`. 

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Secured {
}
```

Then we create an aspect that will add required functionality to `@Secure` annotation. In the aspect, we want to make sure a session exists before the annotated method is invoked. If session does not exist, we throw an exception to indicate user needs to authenticate first. 

```
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;
import shell.security.NotAuthorizedException;
import shell.security.session.SessionHolder;

@Aspect
@Component
public class SecuredAspect {

    private final SessionHolder sessionHolder;

    public SecuredAspect(SessionHolder sessionHolder) {
        this.sessionHolder = sessionHolder;
    }

    @Around("@annotation(Secured)")
    public Object secured(ProceedingJoinPoint joinPoint) throws Throwable {
        if (!sessionHolder.getCurrentClientSession().isPresent()) {
            throw new NotAuthorizedException("First you need to login. Use: login <your login> <your password>");
        }
        return joinPoint.proceed();
    }
}
```

In `SecuredAspect` class, there is usage of `SessionHolder` bean. `sessionHolder` bean is a singleton by default and it holds a session. Here is how such a class could look like. 

```
import java.util.Optional;

public interface SessionHolder {

    void setCurrentSession(Session session);

    Optional<Session> getCurrentClientSession();

    void removeCurrentSession();
}
```

Here is the implementation. 

```
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class DefaultSessionHolder implements SessionHolder {

    private Session session;

    @Override
    public void setCurrentSession(Session session) {
        this.session = session;
    }

    @Override
    public Optional<Session> getCurrentClientSession() {
        return Optional.ofNullable(session);
    }

    @Override
    public void removeCurrentSession() {
        session = null;
    }
}
```

And here the session object. 

```
public class Session {

    private final String authToken;

    public Session(String authToken) {
        this.authToken = authToken;
    }

    public String getAuthToken() {
        return authToken;
    }
}
```

When user login operation is successful, we set a value in the session holder. It could look like this. 

```
import org.springframework.shell.standard.ShellComponent;
import org.springframework.shell.standard.ShellMethod;
import org.springframework.shell.standard.ShellOption;
import shell.features.auth.service.AuthResponse;
import shell.features.auth.service.AuthService;
import shell.security.annotation.Secured;
import shell.security.session.Session;
import shell.security.session.SessionHolder;

import java.util.Optional;

@ShellComponent
public class AuthCommand {

    private final AuthService authService;
    private final SessionHolder sessionHolder;

    public AuthCommand(AuthService authService, SessionHolder sessionHolder) {
        this.authService = authService;
        this.sessionHolder = sessionHolder;
    }

    @ShellMethod(key = "login", value = "Login into GitHub. Usage: login <your login> <your password>")
    public String authenticate(@ShellOption String login,
                               @ShellOption String password) {
        Optional<AuthResponse> response = authService.authenticate(login, password);
        if (response.isPresent()) {
            AuthResponse authResponse = response.get();
            sessionHolder.setCurrentSession(new Session(authResponse.getAuthToken()));
            return "Success.";
        } else {
            return "Failed to authenticate.";
        }
    }

    @Secured
    @ShellMethod(key = "logout", value = "Logout from GitHub. Usage: logout")
    public void logout() {
        sessionHolder.removeCurrentSession();
    }
}
```

You can have a look at the whole example in this [github repository](https://github.com/ondrej-kvasnovsky/github-shell). 

