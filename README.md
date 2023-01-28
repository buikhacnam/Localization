
### `Youtube`: https://www.youtube.com/watch?v=R68OJVIXKv0&ab_channel=CaseyBui
---

- The **`LanguageConfig`** class defines a list of supported locales, in this case "en" and "vi".

```java
@Configuration
public class LanguageConfig {
    public static final List<Locale> LOCALES = Arrays.asList(
            new Locale("en"),
            new Locale("vi")
    );
}
```

- The **`MyLocaleResolver`** class implements the **`LocaleResolver`** interface, which is used to determine the current locale for a request. The **`resolveLocale`** method checks the "Accept-Language" header of the request, and if it is present, it attempts to match it to one of the supported locales defined in the **`LanguageConfig`** class. If a match is found, the corresponding locale is returned. If the header is not present or doesn't match any of the supported locales, the default locale "vi" is returned.

```java
@Component
public class MyLocaleResolver implements LocaleResolver {
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String language = request.getHeader("Accept-Language");
        if (language == null || language.isEmpty()) {
            return Locale.forLanguageTag("vi");
        }
        Locale locale = Locale.forLanguageTag(language);
        if (LanguageConfig.LOCALES.contains(locale)) 
            return locale;
        }
        // return vi if not found
        return Locale.forLanguageTag("vi");
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {
        throw new UnsupportedOperationException("Cannot change locale - use a different locale resolution strategy");
    }
}
```

- The **`MessageConfiguration`** class defines a bean for a **`ResourceBundleMessageSource`** which is used to load message properties files. The files are expected to be named "messages_[locale].properties" and are stored in the classpath.

```java
@Configuration
public class MessageConfiguration {

    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setBasename("messages");
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }
}
```

- The **`TestService`** class is an example of a service that uses the **`MyLocaleResolver`** and **`MessageSource`** beans to return localized messages. The **`message`** method retrieves the current locale for the request and uses the **`messageSource.getMessage`** method to get the localized message for the key "greeting".

```java
@GetMapping("/message")
    public String message(HttpServletRequest request) {
       return testService.message(request);
    }

@Service
public class TestService {

    @Autowired
    private MessageSource messageSource;

    @Autowired
    private MyLocaleResolver myLocaleResolver;

    public String message(HttpServletRequest request) {
        throw new ApiError(400, messageSource.getMessage("greeting", null, myLocaleResolver.resolveLocale(request)));

    }
}
```

```java

// messages_en.properties
greeting=hello

// messages_fr.properties
greeting=bonjour

// messages_vi.properties
greeting=xin chào
```
