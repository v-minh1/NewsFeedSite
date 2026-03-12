
# Upgrade Java Project

## 🖥️ Project Information
- **Project path**: /home/runner/work/NewsFeedSite/NewsFeedSite/service
- **Java version**: 21
- **Build tool type**: Maven
- **Build tool path**: /usr/share/apache-maven-3.9.12/bin

## 🎯 Goals

- Upgrade Java to 21

## 🔀 Changes

### Test Changes
|     | Total | Passed | Failed | Skipped | Errors |
|-----|-------|--------|--------|---------|--------|
| Before | 0 | 0 | 0 | 0 | 0 |
| After | 0 | 0 | 0 | 0 | 0 |
### Dependency Changes


#### Upgraded Dependencies
| Dependency | Original Version | Current Version | Module |
|------------|------------------|-----------------|--------|
| Java | 17 | 21 | Root Module |

#### Added Dependencies
|   Dependency   | Version | Module |
|----------------|---------|--------|
| jakarta.websocket:jakarta.websocket-client-api | 2.2.0 | demo |
| jakarta.websocket:jakarta.websocket-api | 2.2.0 | demo |
| jakarta.servlet:jakarta.servlet-api | 6.1.0 | demo |

#### Removed Dependencies
|   Dependency   | Version | Module |
|----------------|---------|--------|
| javax.websocket:javax.websocket-api | 1.1 | demo |
| javax.servlet:javax.servlet-api | 3.1.0 | demo |

### Code commits

All code changes have been committed to branch `copilot/execute-modernization-plan`, here are the details:
4 files changed, 166 insertions(+), 161 deletions(-)

- 1184f4f -- Upgrade Java 17 to 21, migrate javax.* to jakarta.*, update Maven plugins and Jetty plugin to support Jakarta EE and Java 21

- 3f4dbef -- Fix build: add jakarta.websocket-client-api:2.2.0 for Session and annotation classes

- 5f85ac8 -- fix issues
### Potential Issues

#### CVEs
- junit:junit:4.11:
  - [**MEDIUM**][CVE-2020-15250](https://github.com/advisories/GHSA-269g-pwp5-87pp): TemporaryFolder on unix-like systems does not limit access to created files

- com.rabbitmq:amqp-client:5.16.0:
  - [**MEDIUM**][CVE-2023-46120](https://github.com/advisories/GHSA-mm8h-8587-p46h): RabbitMQ Java client's Lack of Message Size Limitation leads to Remote DoS Attack
