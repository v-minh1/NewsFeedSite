# Modernization Summary: 001-upgrade-java-spring-boot

## Task
Upgrade to Java 21 and Spring Boot 3.4

## Status
✅ **Completed** — Build passes, all tests pass, no CVEs introduced.

## Changes Made

### service/pom.xml
| Change | Before | After |
|--------|--------|-------|
| Java compiler source/target | 17 | 21 |
| `javax.websocket-api` → `jakarta.websocket-client-api` | `javax.websocket:javax.websocket-api:1.1` | `jakarta.websocket:jakarta.websocket-client-api:2.2.0` |
| Added `jakarta.websocket-api` (server-side) | _(not present separately)_ | `jakarta.websocket:jakarta.websocket-api:2.2.0` |
| `javax.servlet-api` → `jakarta.servlet-api` | `javax.servlet:javax.servlet-api:3.1.0` | `jakarta.servlet:jakarta.servlet-api:6.1.0` |
| `maven-surefire-plugin` | 2.22.1 | 3.2.5 |
| Jetty Maven plugin | `org.eclipse.jetty:jetty-maven-plugin:9.4.48.v20220622` | `org.eclipse.jetty.ee10:jetty-ee10-maven-plugin:12.0.21` |

> **Note:** This project does not use Spring Boot or Spring Framework directly. The javax→jakarta namespace migration is equivalent to what Spring Boot 3.x requires and aligns the project with Jakarta EE 10+ standards.

### client/pom.xml
| Change | Before | After |
|--------|--------|-------|
| Java compiler source/target | 17 | 21 |
| `maven-compiler-plugin` | 3.8.0 | 3.13.0 |
| `maven-surefire-plugin` | 2.22.1 | 3.2.5 |

### service/src/main/java/com/example/websocket/NewsWebSocket.java
- Migrated imports from `javax.websocket.*` → `jakarta.websocket.*`
- Migrated imports from `javax.websocket.server.ServerEndpoint` → `jakarta.websocket.server.ServerEndpoint`

### service/src/main/java/com/example/rabbitmq/RabbitMQConsumer.java
- Migrated imports from `javax.servlet.ServletContextEvent` → `jakarta.servlet.ServletContextEvent`
- Migrated imports from `javax.servlet.ServletContextListener` → `jakarta.servlet.ServletContextListener`
- Migrated imports from `javax.servlet.annotation.WebListener` → `jakarta.servlet.annotation.WebListener`

### service/src/main/webapp/WEB-INF/web.xml
- Updated XML namespace from `http://xmlns.jcp.org/xml/ns/javaee` (Servlet 3.1) to `https://jakarta.ee/xml/ns/jakartaee` (Servlet 6.0)
- Updated `xsi:schemaLocation` to use Jakarta EE 6.0 schema
- Updated `version` from `3.1` to `6.0`

## Migration Approach
- Applied OpenRewrite recipe `org.openrewrite.java.migrate.UpgradeToJava21` for the Java 21 compiler upgrade
- Applied OpenRewrite recipe `org.openrewrite.java.migrate.jakarta.JakartaEE10` for the javax→jakarta namespace migration
- Manually added `jakarta.websocket-client-api:2.2.0` because the Jakarta WebSocket API 2.x was split into separate client and server artifacts (unlike the original monolithic `javax.websocket-api:1.1`)

## Validation Results
- ✅ Build: **PASSED**
- ✅ Unit Tests: **PASSED**
- ✅ CVE Check: **No new vulnerabilities introduced**
- ✅ Behavioral Consistency: All javax→jakarta changes are drop-in namespace replacements with no functional behavior change
