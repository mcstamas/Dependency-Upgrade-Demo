# Spring PetClinic Upgrade Report

## Executive Summary

This document details the enterprise-style dependency and framework upgrade performed on the Spring PetClinic application, migrating from Spring Boot 2.7.3 to Spring Boot 3.4.0.

**Upgrade Date:** December 2, 2025  
**Performed By:** Devin AI  
**Final Build Status:** SUCCESS (41 tests passed, 1 skipped)

---

## Version Changes

| Component | Before | After | Notes |
|-----------|--------|-------|-------|
| Java | 1.8 | 17 | LTS version, required for Spring Boot 3.x |
| Spring Boot | 2.7.3 | 3.4.0 | Latest stable release |
| Spring Framework | 5.3.x | 6.2.x | Included with Spring Boot 3.4.0 |
| Hibernate | 5.6.x | 6.6.x | Included with Spring Boot 3.4.0 |
| Tomcat | 9.x | 10.1.x | Jakarta EE 10 compatible |
| Project Version | 2.7.3 | 3.4.0 | Aligned with Spring Boot version |

---

## Dependency Updates

### Core Dependencies (Managed by Spring Boot Parent)

These dependencies are automatically updated through the Spring Boot parent POM:
- Spring Data JPA
- Spring MVC
- Spring Validation
- Thymeleaf
- H2 Database
- PostgreSQL Driver
- Ehcache

### Explicitly Updated Dependencies

| Dependency | Old Version | New Version | Reason |
|------------|-------------|-------------|--------|
| Bootstrap (webjars) | 5.1.3 | 5.3.3 | Latest stable Bootstrap 5 |
| JaCoCo | 0.8.7 | 0.8.12 | Java 17 compatibility |
| Checkstyle | 8.45.1 | 10.18.2 | Java 17 compatibility |
| Spring Format | 0.0.31 | 0.0.43 | Spring Boot 3.x compatibility |
| nohttp-checkstyle | 0.0.10 | 0.0.11 | Latest version |
| maven-checkstyle-plugin | 3.1.2 | 3.5.0 | Latest version |

### Renamed/Replaced Dependencies

| Old Artifact | New Artifact | Reason |
|--------------|--------------|--------|
| `mysql:mysql-connector-java` | `com.mysql:mysql-connector-j` | MySQL connector renamed in 8.0.31+ |
| `pl.project13.maven:git-commit-id-plugin` | `io.github.git-commit-id:git-commit-id-maven-plugin` v9.0.1 | Plugin moved to new coordinates |

### New Dependencies Added

| Dependency | Purpose |
|------------|---------|
| `jakarta.xml.bind:jakarta.xml.bind-api` | JAXB API (removed from JDK 11+) |
| `org.glassfish.jaxb:jaxb-runtime` | JAXB implementation for XML serialization |

---

## Breaking Changes and Resolutions

### 1. Jakarta EE Namespace Migration (Critical)

**Issue:** Spring Boot 3.x requires Jakarta EE 10, which uses the `jakarta.*` namespace instead of `javax.*`.

**Resolution:** Migrated all imports in 16 Java files:

| Package | Old Import | New Import |
|---------|------------|------------|
| JPA | `javax.persistence.*` | `jakarta.persistence.*` |
| Validation | `javax.validation.*` | `jakarta.validation.*` |
| XML Binding | `javax.xml.bind.*` | `jakarta.xml.bind.*` |

**Files Modified:**
- `BaseEntity.java`
- `NamedEntity.java`
- `Person.java`
- `Owner.java`
- `OwnerController.java`
- `Pet.java`
- `PetController.java`
- `PetType.java`
- `Visit.java`
- `VisitController.java`
- `Specialty.java`
- `Vet.java`
- `Vets.java`
- `ValidatorTests.java`

**Note:** `javax.cache` (JCache API/JSR-107) was intentionally NOT migrated as the JCache specification has not adopted the Jakarta namespace.

### 2. JAXB Removal from JDK

**Issue:** JAXB was removed from the JDK in Java 11. The `Vets.java` and `Vet.java` classes use JAXB annotations for XML serialization.

**Resolution:** Added explicit JAXB dependencies:
```xml
<dependency>
    <groupId>jakarta.xml.bind</groupId>
    <artifactId>jakarta.xml.bind-api</artifactId>
</dependency>
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 3. Build Plugin Configuration

**Issue:** The `spring-boot-maven-plugin` build-info configuration referenced `${maven.compiler.source}` and `${maven.compiler.target}` which are no longer automatically set.

**Resolution:** Updated to use `${java.version}` property:
```xml
<java.source>${java.version}</java.source>
<java.target>${java.version}</java.target>
```

### 4. MySQL Connector Artifact Rename

**Issue:** The MySQL connector was renamed from `mysql-connector-java` to `mysql-connector-j` starting with version 8.0.31.

**Resolution:** Updated the dependency coordinates:
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 5. Git Commit ID Plugin Migration

**Issue:** The `git-commit-id-plugin` moved to new Maven coordinates.

**Resolution:** Updated plugin configuration:
```xml
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
    <version>9.0.1</version>
</plugin>
```

---

## Current Warnings and Deprecations

### Compilation Warnings (4 warnings)

The following test files use the deprecated `@MockBean` annotation:
- `OwnerControllerTests.java`
- `PetControllerTests.java`
- `VisitControllerTests.java`
- `VetControllerTests.java`

**Status:** Non-blocking. Tests still function correctly. The `@MockBean` annotation from `org.springframework.boot.test.mock.mockito` is deprecated and marked for removal in future Spring Boot versions.

**Recommended Action:** Migrate to `@MockitoBean` from `org.springframework.test.context.bean.override.mockito` in a future update.

### Runtime Warnings (Thymeleaf Deprecations)

The following Thymeleaf deprecations appear at runtime in `fragments/layout.html`:

1. **Unwrapped Fragment Expressions** (lines 48, 53, 58, 64)
   - Current: `::menuItem ('/','home','home page','home','Home')`
   - Recommended: `~{::menuItem ('/','home','home page','home','Home')}`

2. **Deprecated th:include Attribute** (line 76)
   - Current: `th:include`
   - Recommended: `th:insert`

**Status:** Non-blocking. Templates render correctly but should be updated in a future modernization pass.

### Deprecated API Usage

- `VetTests.java` uses or overrides a deprecated API (non-critical)

---

## Final Build Status

```
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  18.523 s
[INFO] Finished at: 2025-12-02T17:37:29Z
[INFO] ------------------------------------------------------------------------

Tests run: 41, Failures: 0, Errors: 0, Skipped: 1
```

**Build Artifacts:**
- `target/spring-petclinic-3.4.0.jar` - Executable Spring Boot JAR
- `target/spring-petclinic-3.4.0.jar.original` - Original JAR before repackaging

---

## Files Modified Summary

| Category | Files Changed |
|----------|---------------|
| Build Configuration | 2 (pom.xml, build.gradle) |
| Model Classes | 3 (BaseEntity, NamedEntity, Person) |
| Owner Package | 6 (Owner, OwnerController, Pet, PetController, PetType, Visit, VisitController) |
| Vet Package | 3 (Specialty, Vet, Vets) |
| Test Classes | 7 (ValidatorTests, OwnerControllerTests, PetControllerTests, VisitControllerTests, ClinicServiceTests, CrashControllerTests, VetControllerTests) |
| **Total** | **22 files** |

---

## Rollback Instructions

If a rollback is required:

1. Revert to the previous commit:
   ```bash
   git revert HEAD
   ```

2. Or checkout the previous version:
   ```bash
   git checkout 30e16ab
   ```

3. Ensure Java 8 is available in the environment for the older version.
