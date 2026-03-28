# CLAUDE.md Template for Java Projects

Copy this template to the project root and fill in all `[FILL]` placeholders.

```markdown
# Project: [FILL: project name]

## Architecture & Stack
- Java [FILL: version]
- Framework: [FILL: Spring Boot X.X / Spring MVC / plain Java]
- ORM: [FILL: Hibernate X.X / Spring Data JPA / raw JDBC]
- Build: [FILL: Maven / Gradle / Ant]
- VCS: [FILL: Git / CVS+local Git / SVN+local Git]
- Deploy: [FILL: Tomcat / WildFly / JAR / WAR]
- DB: [FILL: Oracle / PostgreSQL / MySQL / H2]

## Commands
- Build: `[FILL: mvn compile / ant compile / gradle build]`
- Test all: `[FILL: mvn test / ant test]`
- Test single: `[FILL: mvn test -Dtest=ClassName / ant test -Dtestcase=ClassName]`
- Package: `[FILL: mvn package / ant dist]`
- Run: `[FILL: mvn spring-boot:run / java -jar target/app.jar]`
- Clean: `[FILL: mvn clean / ant clean]`

## Navigation Map
| Area | Path | Files | Key Classes | Role |
|------|------|-------|-------------|------|
| [FILL] | | | | |

## Code Conventions
- Package naming: [FILL]
- Naming conventions: [FILL: camelCase/Hungarian notation/etc.]
- Test framework: [FILL: JUnit 4 / JUnit 5 / TestNG]
- New tests: JUnit 5 + AssertJ (preferred)
- Existing tests: maintain current framework

## Guardrails
- NEVER modify generated code in [FILL: path if applicable]
- NEVER delete or disable existing tests without approval
- ALWAYS run tests after modifying Java files
- Check N+1 queries when modifying Hibernate/JPA code
- grep for usages before modifying shared utilities
- Limit refactoring to one class per commit
- Large changes require a plan first

## Framework Rules
### Spring
- [FILL: profile names, property file locations, XML vs annotation config]

### Hibernate / JPA
- [FILL: entity naming pattern, HQL vs Criteria, fetch strategy policy]
- Check generated SQL with spring.jpa.show-sql=true when modifying queries

## Testing Policy
- Run full test suite before committing
- If tests fail after your change, fix the CODE not the test
- New code must have corresponding tests following existing patterns
- Integration tests may need DB setup — check test/resources/

## Security
- No hardcoded passwords or API keys
- environment-specific configs (.local, .dev) are NOT tracked in Git
- [FILL: any additional security policies]
```
