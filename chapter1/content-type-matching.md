# Content Type Matching

If we want to sent e.g. email as path variable, Spring Content matching algorithm will think ".com" from the email address is trying to say what should be the content type.

```
@GetMapping(value = "/user/{email}", produces = APPLICATION_JSON_UTF8_VALUE)
```

We can turn this off using the following config.

```
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ContentNegotiationConfigurer;
import org.springframework.web.servlet.config.annotation.PathMatchConfigurer;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class Config implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.setUseSuffixPatternMatch(false);
    }

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.favorPathExtension(false);
    }
}
```



