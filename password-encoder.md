# Password encoders 


## Interface

```java
package example.security;

import io.micronaut.core.annotation.NonNull;
import jakarta.validation.constraints.NotBlank;

public interface IPasswordEncoder {
    String encode(@NotBlank @NonNull String rawPassword);
}
```

## Service

```java
package example.service;

import jakarta.inject.Singleton;
import io.micronaut.core.annotation.NonNull;
import jakarta.validation.constraints.NotBlank;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

import example.security.IPasswordEncoder;

@Singleton
public class BCryptPasswordEncoderService implements IPasswordEncoder {

    private final PasswordEncoder delegate = new BCryptPasswordEncoder();

    @Override
    public String encode(@NotBlank @NonNull String rawPassword) {
        return delegate.encode(rawPassword);
    }

    // Additional method to verify password matches (useful for login)
    public boolean matches(@NotBlank @NonNull String rawPassword,
            @NotBlank @NonNull String encodedPassword) {
        return delegate.matches(rawPassword, encodedPassword);
    }
}
```

## Bean using the encoder service

```java
package example.service;

import io.micronaut.transaction.annotation.Transactional;
import example.domain.User;
import example.exception.UserAlreadyExistsException;
import example.repository.UserRepository;
import example.security.PasswordEncoder;
import io.micronaut.core.annotation.NonNull;
import jakarta.inject.Singleton;

@Singleton
public class RegistrationService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public RegistrationService(UserRepository userRepository, 
                               PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Transactional
    public User registerUser(@NonNull String username, @NonNull String rawPassword) {
        // Check if user already exists
        if (userRepository.findByUsername(username).isPresent()) {
            throw new UserAlreadyExistsException("Username already exists: " + username);
        }
        
        // Hash the password using the injected encoder
        String hashedPassword = passwordEncoder.encode(rawPassword);
        
        User user = new User();
        user.setUsername(username);
        user.setPassword(hashedPassword);
        
        return userRepository.save(user);
    }
}
```
