# Post-Upgrade Validation Checklist

This checklist provides step-by-step instructions for validating the Spring PetClinic application after the Spring Boot 3.4.0 upgrade.

---

## Prerequisites

Before running the application, ensure you have:

- [ ] Java 17 or later installed (`java -version` should show 17.x.x or higher)
- [ ] Maven 3.8+ (or use the included Maven wrapper `./mvnw`)
- [ ] Git (for version control operations)

---

## Quick Start Validation

### 1. Build Verification

Run the full build to ensure everything compiles and tests pass:

```bash
cd ~/repos/Dependency-Upgrade-Demo
./mvnw clean package
```

**Expected Result:**
- BUILD SUCCESS
- Tests run: 41, Failures: 0, Errors: 0, Skipped: 1
- JAR file created at `target/spring-petclinic-3.4.0.jar`

### 2. Start the Application

```bash
./mvnw spring-boot:run
```

**Expected Result:**
- Application starts without errors
- Banner displays "Built with Spring Boot :: 3.4.0"
- Server starts on port 8080
- Log shows: "Started PetClinicApplication in X.XXX seconds"

### 3. Access the Application

Open a web browser and navigate to:
- **Home Page:** http://localhost:8080
- **Find Owners:** http://localhost:8080/owners/find
- **Veterinarians:** http://localhost:8080/vets.html
- **Error Page:** http://localhost:8080/oups

---

## Functional Validation Checklist

### Home Page
- [ ] Home page loads without errors
- [ ] Navigation menu displays correctly
- [ ] PetClinic logo and styling appear properly
- [ ] All navigation links are functional

### Owner Management
- [ ] **Find Owners:** Search form displays at `/owners/find`
- [ ] **Search:** Empty search returns all owners
- [ ] **Search:** Searching for "Davis" returns Betty and Harold Davis
- [ ] **View Owner:** Click on an owner shows their details and pets
- [ ] **Add Owner:** Create a new owner at `/owners/new`
  - Fill in: First Name, Last Name, Address, City, Telephone
  - Submit and verify redirect to owner details
- [ ] **Edit Owner:** Modify an existing owner's information
- [ ] **Pagination:** Owner list pagination works correctly

### Pet Management
- [ ] **Add Pet:** Add a new pet to an owner
  - Select pet type from dropdown (cat, dog, bird, etc.)
  - Enter name and birth date
  - Submit and verify pet appears in owner details
- [ ] **Edit Pet:** Modify an existing pet's information
- [ ] **Pet Types:** All pet types load correctly in dropdown

### Visit Management
- [ ] **Add Visit:** Add a visit record for a pet
  - Enter date and description
  - Submit and verify visit appears in pet's history
- [ ] **Visit History:** Previous visits display correctly

### Veterinarians
- [ ] **Vet List (HTML):** `/vets.html` displays paginated vet list
- [ ] **Vet List (JSON):** `/vets` returns valid JSON response
- [ ] **Specialties:** Vet specialties display correctly
- [ ] **Pagination:** Vet list pagination works

### Error Handling
- [ ] **Error Page:** `/oups` triggers and displays error page correctly
- [ ] **404 Handling:** Non-existent URLs show appropriate error

---

## Database Validation

### H2 Console (Development)

Access the H2 database console:
1. Navigate to: http://localhost:8080/h2-console
2. Use these settings:
   - JDBC URL: `jdbc:h2:mem:testdb`
   - Username: `sa`
   - Password: (leave empty)
3. Click "Connect"

**Verify:**
- [ ] Tables exist: `owners`, `pets`, `visits`, `vets`, `specialties`, `types`, `vet_specialties`
- [ ] Sample data is loaded (6 owners, 13 pets, 4 visits, 6 vets)

### MySQL Profile (Optional)

To test with MySQL:

1. Start MySQL using Docker:
   ```bash
   docker-compose up -d mysql
   ```

2. Run with MySQL profile:
   ```bash
   ./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql
   ```

3. Verify application connects and functions correctly

### PostgreSQL Profile (Optional)

To test with PostgreSQL:

1. Start PostgreSQL using Docker:
   ```bash
   docker-compose up -d postgres
   ```

2. Run with PostgreSQL profile:
   ```bash
   ./mvnw spring-boot:run -Dspring-boot.run.profiles=postgres
   ```

3. Verify application connects and functions correctly

---

## API Validation

### Actuator Endpoints

Spring Boot Actuator endpoints are available at `/actuator`:

- [ ] **Health:** `GET /actuator/health` returns `{"status":"UP"}`
- [ ] **Info:** `GET /actuator/info` returns build information
- [ ] **Metrics:** `GET /actuator/metrics` lists available metrics

### REST Endpoints

- [ ] **Vets JSON:** `GET /vets` returns JSON array of veterinarians
- [ ] **Vets XML:** `GET /vets` with `Accept: application/xml` returns XML (JAXB)

Test with curl:
```bash
# JSON response
curl -H "Accept: application/json" http://localhost:8080/vets

# XML response
curl -H "Accept: application/xml" http://localhost:8080/vets
```

---

## Caching Validation

The application uses JCache (Ehcache) for caching veterinarian data.

### Verify Cache Operation

1. Access `/vets.html` - this populates the cache
2. Check cache statistics via JMX or actuator metrics
3. Access `/vets.html` again - should be served from cache

**Expected:** Second request should be faster (cache hit)

---

## Performance Validation

### Startup Time

- [ ] Application starts in under 5 seconds (typical: 2-3 seconds)
- [ ] No excessive warnings during startup

### Memory Usage

- [ ] Application runs with default heap settings
- [ ] No OutOfMemoryError during normal operation

---

## Known Warnings (Non-Blocking)

The following warnings are expected and do not affect functionality:

### Compilation Warnings
```
@MockBean in org.springframework.boot.test.mock.mockito has been deprecated
```
**Impact:** None. Tests function correctly.

### Runtime Warnings (Thymeleaf)
```
Deprecated unwrapped fragment expression found in template fragments/layout
Deprecated attribute {th:include,data-th-include} found
```
**Impact:** None. Templates render correctly.

---

## Troubleshooting

### Application Won't Start

1. **Check Java version:**
   ```bash
   java -version
   ```
   Must be Java 17 or later.

2. **Check port availability:**
   ```bash
   lsof -i :8080
   ```
   Kill any process using port 8080.

3. **Clean and rebuild:**
   ```bash
   ./mvnw clean package -DskipTests
   ```

### Tests Failing

1. **Run tests with details:**
   ```bash
   ./mvnw test -X
   ```

2. **Run specific test:**
   ```bash
   ./mvnw test -Dtest=OwnerControllerTests
   ```

### Database Connection Issues

1. **H2:** Ensure no other instance is running
2. **MySQL/PostgreSQL:** Verify Docker containers are running:
   ```bash
   docker-compose ps
   ```

---

## Sign-Off

| Validation Area | Status | Tester | Date |
|-----------------|--------|--------|------|
| Build & Tests | | | |
| Home Page | | | |
| Owner CRUD | | | |
| Pet CRUD | | | |
| Visit CRUD | | | |
| Veterinarians | | | |
| Error Handling | | | |
| Database (H2) | | | |
| Actuator | | | |
| Caching | | | |

**Final Approval:**

- [ ] All critical functionality verified
- [ ] No blocking issues identified
- [ ] Application approved for deployment

Approved By: _________________ Date: _________________
