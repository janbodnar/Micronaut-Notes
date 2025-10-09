# Micronaut CSRF Protection Implementation Guide

## Summary

This document explains how to implement CSRF (Cross-Site Request Forgery) protection in a Micronaut application using the **Double Submit Cookie Pattern** with Handlebars templates.

## Final Working Solution

### 1. Dependencies (build.gradle)

```gradle
dependencies {
    annotationProcessor("io.micronaut:micronaut-http-validation")
    annotationProcessor("io.micronaut.security:micronaut-security-annotations")
    annotationProcessor("io.micronaut.serde:micronaut-serde-processor")
    implementation("io.micronaut.security:micronaut-security-session")
    implementation("io.micronaut.serde:micronaut-serde-jackson")
    implementation("io.micronaut.views:micronaut-views-handlebars")
    implementation("io.micronaut.security:micronaut-security-csrf")
    compileOnly("io.micronaut:micronaut-http-client")
    runtimeOnly("ch.qos.logback:logback-classic")
    runtimeOnly("org.yaml:snakeyaml")
    testImplementation("io.micronaut:micronaut-http-client")
}
```

### 2. Configuration (application.yml)

```yaml
micronaut:
  application:
    name: micronautCsrfPebbleExample
  server:
    port: 8080
  security:
    enabled: true
    session:
      enabled: true  # Enable session-based security
    csrf:
      enabled: true
      field-name: csrfToken  # Default name for the form field
      header-name: X-CSRF-TOKEN  # Default header for AJAX
      http-session-name: csrfToken  # Key in HTTP Session
```

### 3. Controller (HomeController.java)

```java
package com.example;

import io.micronaut.http.HttpRequest;
import io.micronaut.http.HttpResponse;
import io.micronaut.http.MediaType;
import io.micronaut.http.MutableHttpResponse;
import io.micronaut.http.annotation.Body;
import io.micronaut.http.annotation.Controller;
import io.micronaut.http.annotation.Get;
import io.micronaut.http.annotation.Post;
import io.micronaut.http.cookie.Cookie;
import io.micronaut.security.annotation.Secured;
import io.micronaut.security.csrf.CsrfConfiguration;
import io.micronaut.security.csrf.generator.CsrfTokenGenerator;
import io.micronaut.security.rules.SecurityRule;
import io.micronaut.views.View;
import jakarta.inject.Inject;

import java.util.HashMap;
import java.util.Map;

@Controller("/")
@Secured(SecurityRule.IS_ANONYMOUS)
public class HomeController {

    @Inject
    CsrfTokenGenerator<HttpRequest<?>> csrfTokenGenerator;

    @Inject
    CsrfConfiguration csrfConfiguration;

    @Get(produces = MediaType.TEXT_HTML)
    @View("home")
    HttpResponse<Map<String, Object>> showForm(HttpRequest<?> request) {
        Map<String, Object> model = new HashMap<>();
        
        // Generate CSRF token
        String csrfToken = csrfTokenGenerator.generateCsrfToken(request);
        
        // Add token to model for the view
        model.put("csrfToken", csrfToken);
        
        // Create response with CSRF token in cookie (following Micronaut's pattern)
        MutableHttpResponse<Map<String, Object>> response = HttpResponse.ok(model);
        Cookie cookie = Cookie.of(csrfConfiguration.getCookieName(), csrfToken);
        cookie.configure(csrfConfiguration, request.isSecure());
        response.cookie(cookie);
        
        return response;
    }

    @Post(consumes = MediaType.APPLICATION_FORM_URLENCODED)
    public HttpResponse<Map<String, String>> save(@Body Map<String, String> data) {
        return HttpResponse.ok(data);
    }
}
```

### 4. Template (home.hbs)

```handlebars
<!DOCTYPE html>
<html>
<head>
    <title>CSRF Form</title>
</head>
<body>
<h1>CSRF Form</h1>
<form action="/" method="post">
    <input type="hidden" name="csrfToken" value="{{csrfToken}}"/>
    <label for="name">Name:</label>
    <input type="text" id="name" name="name"/>
    <button type="submit">Submit</button>
</form>
</body>
</html>
```

## How It Works

### Double Submit Cookie Pattern

1. **Token Generation**: When a user requests the form (GET request), the controller:
   - Generates a unique CSRF token using `CsrfTokenGenerator`
   - Stores the token in a cookie using proper Micronaut configuration
   - Passes the same token to the view template

2. **Token Rendering**: The Handlebars template renders the token in a hidden form field:
   ```html
   <input type="hidden" name="csrfToken" value="{{csrfToken}}"/>
   ```

3. **Token Validation**: When the form is submitted (POST request):
   - The browser sends both the cookie AND the form field with the token
   - Micronaut's `CsrfFilter` automatically extracts both tokens
   - It validates that the cookie value matches the form field value
   - If they match, the request proceeds; otherwise, it returns 401 Unauthorized

## Key Components

### CsrfTokenGenerator<T>
- **Generic Type**: Must be parameterized with `HttpRequest<?>`
- **Method**: `generateCsrfToken(HttpRequest<?> request)` - generates a unique token

### CsrfConfiguration
- Provides all CSRF-related configuration values
- Methods:
  - `getCookieName()` - returns the configured cookie name
  - Used by `cookie.configure()` to apply all security settings

### Cookie Configuration
The `cookie.configure(csrfConfiguration, request.isSecure())` method applies:
- Cookie name (from configuration)
- HttpOnly flag
- Secure flag (based on HTTPS)
- SameSite attribute
- Path and domain settings
- Max age

### CsrfFilter
- Built-in Micronaut filter that runs on POST/PUT/PATCH/DELETE requests
- Automatically validates CSRF tokens
- Looks for tokens in:
  - Form field (name specified in `field-name` configuration)
  - HTTP Header (name specified in `header-name` configuration)
- Compares against the cookie value

## Important Notes

1. **Template Engine**: We used Handlebars instead of Pebble for better compatibility
   - Handlebars syntax: `{{csrfToken}}`
   - Pebble syntax would be: `{{ csrfToken }}`

2. **Generic Types**: Always parameterize `CsrfTokenGenerator` properly:
   ```java
   CsrfTokenGenerator<HttpRequest<?>> csrfTokenGenerator;
   ```

3. **Cookie Creation**: Follow Micronaut's pattern from `CsrfLoginCookieProvider`:
   ```java
   Cookie cookie = Cookie.of(csrfConfiguration.getCookieName(), csrfToken);
   cookie.configure(csrfConfiguration, request.isSecure());
   ```

4. **Field Name**: Ensure the form field name matches the configuration:
   - Configuration: `field-name: csrfToken`
   - HTML: `<input type="hidden" name="csrfToken" .../>`

5. **Session Support**: Enable sessions in configuration for proper CSRF token management:
   ```yaml
   micronaut.security.session.enabled: true
   ```

## Testing

To test the implementation:

1. Start the application: `./gradlew run`
2. Navigate to `http://localhost:8080`
3. View page source - you should see the CSRF token in the hidden field:
   ```html
   <input type="hidden" name="csrfToken" value=".T3S2tH7mBfgh_Legh_p1qw"/>
   ```
4. Submit the form - it should succeed (200 OK)
5. Try submitting without the token or with a wrong token - it should fail (401 Unauthorized)

## Security Benefits

- **Prevents CSRF Attacks**: Ensures that form submissions come from legitimate pages
- **Stateless Validation**: Uses cookies, no server-side session storage required (when using cookie-based approach)
- **Automatic Protection**: Micronaut's filter handles validation automatically
- **Configurable**: Easy to customize cookie settings, field names, etc.

## References

- [Micronaut Security CSRF Documentation](https://micronaut-projects.github.io/micronaut-security/latest/guide/#csrf)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Double Submit Cookie Pattern](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#double-submit-cookie)
