# CLI commands 


## 🚀 Project Creation & Setup (with Language, Build, and Test Options)

| Command | Description |
|--------|-------------|
| `mn create-app <name>` | Create a new Micronaut app |
| `mn create-app <name> --lang kotlin` | Use Kotlin |
| `mn create-app <name> --lang groovy` | Use Groovy |
| `mn create-app <name> --build maven` | Use Maven |
| `mn create-app <name> --build gradle` | Use Gradle (default) |
| `mn create-app <name> --test junit` | Use JUnit |
| `mn create-app <name> --test spock` | Use Spock |
| `mn create-app <name> --features <f1,f2>` | Add specific features |
| `mn create-cli-app <name>` | Create a CLI app |
| `mn create-function <name>` | Create a serverless function |
| `mn create-federation <name>` | Create a federation of services |
| `mn create-profile <name>` | Create a custom CLI profile |
| `mn create-app --list-features` | List all available features |
| `mn create-app --lang java --build gradle --test junit` | Full setup with Java, Gradle, JUnit |

---

## 📦 Feature & Profile Management

| Command | Description |
|--------|-------------|
| `mn list-profiles` | List available profiles |
| `mn profile-info <profile>` | Show profile details |
| `mn feature-diff --features <f>` | Show changes from a feature |
| `mn create-app --features kafka,security-jwt` | Add multiple features |
| `mn create-app --lang kotlin --features data-jdbc,h2` | Kotlin + DB features |
| `mn create-app --features graalvm` | Add GraalVM support |
| `mn create-app --features openapi` | Add OpenAPI support |
| `mn create-app --features swagger-ui` | Add Swagger UI |
| `mn create-app --features micrometer-prometheus` | Add Prometheus metrics |
| `mn create-app --features tracing-jaeger` | Add Jaeger tracing |

---

## 🧱 Component Generation

| Command | Description |
|--------|-------------|
| `mn create-controller <name>` | Generate a controller |
| `mn create-client <name>` | Generate an HTTP client |
| `mn create-service <name>` | Generate a service class |
| `mn create-job <name>` | Generate a scheduled job |
| `mn create-bean <name>` | Generate a singleton bean |
| `mn create-filter <name>` | Generate an HTTP filter |
| `mn create-interceptor <name>` | Generate an AOP interceptor |
| `mn create-entity <name>` | Generate a JPA entity |
| `mn create-config <name>` | Generate a config class |
| `mn create-properties <name>` | Generate a properties file |
| `mn create-kafka-consumer <name>` | Generate a Kafka consumer |
| `mn create-kafka-producer <name>` | Generate a Kafka producer |
| `mn create-factory <name>` | Generate a bean factory |
| `mn create-record <name>` | Generate a Java record |
| `mn create-function-bean <name>` | Generate a function bean |
| `mn create-grpc-service <name>` | Generate a gRPC service |
| `mn create-websocket <name>` | Generate a WebSocket endpoint |
| `mn create-openapi-client <url>` | Generate client from OpenAPI |
| `mn create-reactive-repository <name>` | Generate reactive repo |
| `mn create-rest-controller <name>` | REST controller shortcut |

---

## 🧪 Testing Utilities

| Command | Description |
|--------|-------------|
| `mn create-test <name>` | Generate a test class |
| `mn create-spec <name>` | Generate a Spock spec |
| `mn create-junit-test <name>` | Generate a JUnit test |
| `mn create-validator <name>` | Generate a custom validator |
| `mn create-mock <name>` | Generate a mock bean |
| `mn create-test-resources <name>` | Generate test resources |
| `mn create-test-container <name>` | Generate test container config |
| `mn create-test-factory <name>` | Generate test factory |
| `mn create-test-suite <name>` | Generate a test suite |
| `mn create-test-client <name>` | Generate test HTTP client |

---

## ⚙️ Configuration & Environment

| Command | Description |
|--------|-------------|
| `mn create-config <name>` | Generate a config class |
| `mn create-properties <name>` | Generate a properties file |
| `mn create-environment <name>` | Create a new environment profile |
| `mn create-banner` | Generate a custom Micronaut banner |
| `mn create-bootstrap <name>` | Generate bootstrap config |
| `mn create-yaml <name>` | Generate YAML config |
| `mn create-secret <name>` | Generate a secret config |
| `mn create-env-loader <name>` | Generate env loader class |
| `mn create-application-yml` | Generate application.yml |
| `mn create-application-json` | Generate application.json |

---

## 🛠️ General Utilities & Diagnostics

| Command | Description |
|--------|-------------|
| `mn help` | Show CLI help |
| `mn --version` | Show CLI version |
| `mn --stacktrace` | Show full stack trace |
| `mn --verbose` | Enable verbose output |
| `mn --plain-output` | Disable ANSI formatting |
| `mn create-app --build gradle_kotlin` | Use Gradle Kotlin DSL |
| `mn create-app --lang scala` | Use Scala (if supported) |
| `mn create-app --jdk 17` | Target specific JDK version |
| `mn create-app --mainClass <fqcn>` | Set custom main class |
| `mn create-app --defaultPackage <pkg>` | Set base package |
| `mn create-app --inplace` | Generate app in current directory |
| `mn create-app --no-git` | Skip Git init |
| `mn create-app --no-wrapper` | Skip Gradle wrapper |
| `mn create-app --build-native-image` | Build native image |
| `mn create-app --docker` | Add Docker support |
| `mn create-app --features security-oauth2` | Add OAuth2 security |

---

That’s your **Micronaut CLI power list**—from zero to hero in 100 commands. Want a custom script that scaffolds a full-stack app with your preferred stack? I can whip that up too.
