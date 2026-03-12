# Upgrade Plan

## Overview
Upgrade to the latest LTS versions: Java 21 and Spring Boot 3.4.

## Tasks
See `tasks.json` for detailed task breakdown.

### 001 - Upgrade Java & Spring Boot
- **Target Java**: 21 (LTS)
- **Target Spring Boot**: 3.4
- **Target Spring Framework**: 6.x
- Migrate `javax.*` to `jakarta.*` namespace as required
- Update all Spring ecosystem dependencies to Spring Boot 3.4-compatible versions
- Update build tooling plugins (Maven/Gradle) for Java 21 support
