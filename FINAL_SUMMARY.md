# Spring PetClinic Dependency Upgrade - Engineering Handoff Summary

**Date:** December 2, 2025  
**Upgrade Performed By:** Devin AI  
**PR:** [#4](https://github.com/mcstamas/Dependency-Upgrade-Demo/pull/4)

## Executive Summary

This document summarizes the enterprise-style dependency and framework upgrade performed on the Spring PetClinic application. The project was upgraded from Spring Boot 3.4.0 to 3.5.8, with all breaking changes resolved and tests passing.

## What Was Upgraded

| Component | Previous Version | New Version | Notes |
|-----------|-----------------|-------------|-------|
| Spring Boot | 3.4.0 | 3.5.8 | Latest stable release |
| Java | 17 | 17 | No change (system constraint) |
| JaCoCo | 0.8.12 | 0.8.14 | Code coverage plugin |
| Spring Format | 0.0.43 | 0.0.47 | Code formatting plugin |
| Checkstyle | 10.18.2 | 10.21.4 | Code style enforcement |

### Why These Upgrades

Spring Boot 3.5.8 includes security patches, performance improvements, and bug fixes over 3.4.0. Keeping dependencies current reduces technical debt and ensures compatibility with the broader Spring ecosystem. The plugin updates ensure tooling compatibility with the new Spring Boot version.

## Breaking Changes Resolved

### 1. MockBean Annotation Migration

**Issue:** `org.springframework.boot.test.mock.mockito.MockBean` is deprecated and marked for removal in Spring Boot 3.5.x.

**Resolution:** Migrated to `org.springframework.test.context.bean.override.mockito.MockitoBean` in all test files.

**Files Changed:**
- `src/test/java/.../owner/OwnerControllerTests.java`
- `src/test/java/.../owner/PetControllerTests.java`
- `src/test/java/.../owner/VisitControllerTests.java`
- `src/test/java/.../vet/VetControllerTests.java`

### 2. Thymeleaf Fragment Expression Syntax

**Issue:** Unwrapped fragment expressions (e.g., `::menuItem(...)`) are deprecated in Thymeleaf 3.1+.

**Resolution:** Updated to complete fragment expression syntax using `~{::menuItem(...)}`.

**File Changed:** `src/main/resources/templates/fragments/layout.html`

### 3. Thymeleaf th:include Deprecation

**Issue:** The `th:include` attribute is deprecated in favor of `th:insert`.

**Resolution:** Replaced `th:include="${template}"` with `th:insert="${template}"`.

**File Changed:** `src/main/resources/templates/fragments/layout.html`

## Pre-Existing State (No Changes Required)

The following items were already addressed in the codebase prior to this upgrade:

- **Jakarta EE Migration:** The project was already using Jakarta namespace (`jakarta.xml.bind-api`, `jakarta.persistence`, etc.) - no `javax.*` to `jakarta.*` migration was needed.
- **JAXB Dependencies:** Already included `jakarta.xml.bind-api` and `jaxb-runtime` for XML serialization support.
- **Spring Data JPA:** Already compatible with Spring Boot 3.x.

## New Warnings and Deprecated Areas Detected

### Currently Active Deprecations

1. **VetTests.java** - Uses deprecated API (minor, serialization-related)
   - Location: `src/test/java/.../vet/VetTests.java`
   - Severity: Low
   - Action: Review and update in next maintenance cycle

### Potential Future Deprecations to Monitor

1. **Thymeleaf Layout Dialect** - Monitor for changes in Thymeleaf 4.x
2. **Spring Security** - Not currently used, but if added, ensure 6.x compatibility
3. **Hibernate** - Monitor for ORM changes in future Spring Boot releases

## Remaining Modernization Opportunities

### Code Quality

1. **Test Coverage:** Current JaCoCo coverage should be reviewed; consider setting minimum thresholds
2. **Static Analysis:** Consider adding SpotBugs or SonarQube integration
3. **Dependency Vulnerability Scanning:** Add OWASP dependency-check plugin

### Architecture

1. **Java 21 Upgrade:** When infrastructure supports it, upgrade to Java 21 for virtual threads and pattern matching
2. **Native Compilation:** Consider GraalVM native image for faster startup
3. **Observability:** Add Micrometer tracing for distributed tracing support

---

## Next Steps for Engineering Team

### Immediate Follow-up Tasks (This Sprint)

- [ ] **Verify Production Deployment:** Deploy to staging environment and run smoke tests
- [ ] **Review VetTests Deprecation:** Update `VetTests.java` to remove deprecated API usage
- [ ] **Update CI/CD Pipeline:** Ensure build pipelines use compatible Maven/Java versions

### Medium-term Modernization Items (Next 1-3 Months)

- [ ] **Add Dependency Vulnerability Scanning:** Integrate OWASP dependency-check or Snyk
- [ ] **Implement Code Coverage Thresholds:** Set JaCoCo minimum coverage requirements
- [ ] **Review Thymeleaf Templates:** Audit all templates for deprecated syntax patterns
- [ ] **Add Integration Tests:** Expand test coverage for controller endpoints
- [ ] **Document API Endpoints:** Consider adding OpenAPI/Swagger documentation

### Long-term Architectural Recommendations (3-12 Months)

- [ ] **Java 21 Migration:** Plan upgrade path when infrastructure supports it
  - Benefits: Virtual threads, pattern matching, record patterns
  - Effort: Low (mostly configuration changes)

- [ ] **Consider Reactive Stack:** Evaluate Spring WebFlux for high-concurrency scenarios
  - Benefits: Better resource utilization under load
  - Effort: High (significant code changes)

- [ ] **Native Image Compilation:** Evaluate GraalVM native compilation
  - Benefits: Sub-second startup, reduced memory footprint
  - Effort: Medium (reflection configuration, testing)

- [ ] **Modular Architecture:** Consider breaking into microservices if scaling needs arise
  - Benefits: Independent deployment, technology flexibility
  - Effort: High (architectural redesign)

- [ ] **Database Migration:** Evaluate moving from H2 to production-grade database for non-dev environments
  - Current: H2 in-memory (dev), MySQL/PostgreSQL (prod profiles exist)
  - Action: Ensure production profiles are properly configured and tested

---

## Build Verification

```bash
# Full build with tests
./mvnw clean package

# Results: BUILD SUCCESS
# Tests: 41 run, 0 failures, 0 errors, 1 skipped
```

## References

- [Spring Boot 3.5 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.5-Release-Notes)
- [Spring Framework 6.2 Documentation](https://docs.spring.io/spring-framework/reference/)
- [Thymeleaf Documentation](https://www.thymeleaf.org/documentation.html)
- [MockitoBean Migration Guide](https://docs.spring.io/spring-framework/reference/testing/annotations/integration-spring/annotation-mockitobean.html)
