# Legacy Java Patterns Catalog

Extended detection patterns with exact Claude Code fix prompts.

## High Severity

### H1: N+1 Query (EAGER Fetch)
- **Detect**: `grep -rn "FetchType.EAGER" --include="*.java"`
- **Impact**: Loading N entities triggers N+1 SQL queries. 100 customers = 101 queries
- **Fix prompt**: `"Change FetchType.EAGER to FetchType.LAZY on [entity.field]. Add @EntityGraph to the repository method that loads this relationship. Run tests to verify."`

### H2: Entity Exposed in REST API
- **Detect**: Controller methods returning @Entity classes directly (no DTO/response wrapper)
- **Impact**: Exposes DB structure, internal IDs, lazy-load proxies. Security risk + serialization errors
- **Fix prompt**: `"Create a DTO class for [Entity] with only the fields needed by the API. Add a mapper method. Update the controller to return the DTO instead of the entity. Run tests."`

### H3: No Tests for Critical Module
- **Detect**: `find src/main/java/[package] -name "*.java" | wc -l` vs `find src/test/java/[package] -name "*Test*.java" | wc -l`
- **Impact**: Cannot verify changes safely. Claude Code's build→test→fix loop is broken
- **Fix prompt**: `"Generate JUnit tests for [ClassName] covering all public methods. Include happy path, null input, and boundary cases. Follow existing test patterns in the project."`

## Medium Severity

### M1: Field Injection (@Autowired on fields)
- **Detect**: `grep -rn "@Autowired" --include="*.java" | grep -v "constructor\|public.*("`
- **Impact**: Hard to test (requires Spring context), hides dependencies, allows circular deps
- **Fix prompt**: `"Refactor [ClassName] from field injection to constructor injection. Remove @Autowired from fields, add a constructor that takes all dependencies, add final to the fields. Run tests."`

### M2: java.util.Date Usage
- **Detect**: `grep -rn "import java.util.Date" --include="*.java"`
- **Impact**: Not thread-safe, mutable, deprecated API. Timezone handling is error-prone
- **Fix prompt**: `"Migrate [ClassName] from java.util.Date to java.time.LocalDateTime (or ZonedDateTime if timezone matters). Update the entity @Temporal annotation to @Column with appropriate type. Run tests."`
- **Note**: For Spring Boot 2.x (Java 8+), java.time is fully supported. For entities, may need `@Column(columnDefinition = "TIMESTAMP")`.

### M3: Magic Numbers (Status/Type Codes)
- **Detect**: Hardcoded int values for status (0,1,2,9) or type (1,2,3)
- **Impact**: Code is unreadable. `if (status == 9)` means nothing without context
- **Fix prompt**: `"Extract the magic numbers for [field] in [ClassName] into a Java enum. Replace all usages of the raw int with the enum. For JPA entities, add @Enumerated(EnumType.STRING) or use an AttributeConverter. Run tests."`

### M4: Raw List (No Generics)
- **Detect**: `List results = new ArrayList()` without type parameter, `(Customer) it.next()` casts
- **Impact**: ClassCastException at runtime, no compile-time type safety
- **Fix prompt**: `"Add generic type parameters to all raw List/Map/Set usages in [ClassName]. Remove manual casts. Run tests."`

### M5: Manual JDBC (No ORM)
- **Detect**: `grep -rn "DriverManager\|PreparedStatement\|ResultSet" --include="*.java"`
- **Impact**: Boilerplate, resource leak risk, SQL injection if not using PreparedStatement
- **Fix prompt**: `"Migrate [DaoClass] from raw JDBC to Spring Data JPA. Create a corresponding @Entity and @Repository interface. Verify the generated SQL matches the original queries. Run tests."`

### M6: Broad Exception Handling
- **Detect**: `catch (Exception e)` in service/controller methods
- **Impact**: Swallows specific exceptions, hides bugs
- **Fix prompt**: `"Replace the broad catch(Exception) in [method] with specific exception types. Create custom exceptions where appropriate (e.g., AccountNotFoundException). Run tests."`

## Low Severity

### L1: javax.persistence (Pre-Jakarta)
- **Detect**: `grep -rn "import javax.persistence" --include="*.java"`
- **Impact**: Indicates Spring Boot 2.x. Not broken, but blocks upgrade to Spring Boot 3+
- **Fix prompt**: `"Migrate javax.persistence imports to jakarta.persistence. This is a prerequisite for Spring Boot 3.x upgrade. Run tests."`
- **Note**: Can be automated with OpenRewrite: `mvn rewrite:run -Drewrite.activeRecipes=org.openrewrite.java.migrate.jakarta.JakartaEE10`

### L2: Missing Javadoc on Public API
- **Detect**: Public methods in service/controller without Javadoc
- **Impact**: Makes codebase harder to navigate, especially for new team members
- **Fix prompt**: `"Add Javadoc to all public methods in [ClassName]. Include @param, @return, and @throws tags. Keep descriptions concise."`

### L3: Deprecated API Usage
- **Detect**: Compiler warnings about deprecated APIs
- **Impact**: Will break on future Java/framework upgrades
- **Fix prompt**: `"Find and replace all deprecated API usages in [ClassName] with their modern equivalents. Check the Javadoc of each deprecated method for the recommended replacement. Run tests."`
