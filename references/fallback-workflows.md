# Fallback Workflows for Lv1/Lv2 Environments

When CLI build or test is not available, Claude Code still provides value — but the workflow changes.

## Lv2: CLI Build Works, Test is Manual

**Situation**: `mvn compile` works, but `mvn test` fails (needs DB, Eclipse runner, or special setup).

**Workflow**:
```
1. Claude Code modifies code
2. Claude Code runs `mvn compile` → confirms it compiles
3. Claude Code commits with message "[needs-test] description"
4. Human runs tests in Eclipse / IDE
5. If tests fail → human describes failure to Claude Code → Claude Code fixes
6. If tests pass → done
```

**CLAUDE.md addition**:
```markdown
## Testing
- CLI test is NOT available. Do NOT run `mvn test`.
- After modifying code, run `mvn compile` to verify compilation.
- Tag commits with [needs-test] for human verification.
- When told "test X failed with error Y", analyze and fix.
```

**Value at Lv2**: Claude Code handles ~70% of the work (analysis, modification, compilation). Human handles test verification.

---

## Lv1: Build Requires IDE (Eclipse Only)

**Situation**: No CLI build at all. Eclipse's internal builder is the only way to compile.

**Workflow**:
```
1. Claude Code analyzes codebase (read-only)
2. Claude Code generates:
   - Navigation Map / architecture documentation
   - Code review with anti-pattern detection
   - Refactoring proposals (as diff suggestions)
   - Test generation (human pastes into IDE and runs)
   - Documentation from undocumented code
3. Human applies changes in Eclipse
4. Human builds and tests in Eclipse
5. Human reports results to Claude Code for iteration
```

**CLAUDE.md addition**:
```markdown
## Build
- CLI build is NOT available. Do NOT attempt to run build commands.
- This is a read-only analysis environment.
- Generate code changes as suggestions, not direct edits.
- Focus on: analysis, documentation, code review, refactoring proposals.
```

**Value at Lv1**:
- Architecture documentation (huge value for undocumented legacy systems)
- Code review / anti-pattern detection
- Test code generation (human copies to IDE)
- Refactoring plan creation
- Knowledge extraction from "tribal knowledge" code

**Key insight**: Even at Lv1, Claude Code understanding 500k lines of undocumented code and producing a Navigation Map + architecture document is worth days of human effort.

---

## Lv1 → Lv2 Upgrade Path

If stuck at Lv1, the priority is getting to Lv2. Steps:

1. **Identify what Eclipse does that CLI can't**
   ```
   Ask the team: "What happens when you click Build in Eclipse?"
   Record every step. Look for:
   - Code generation (annotation processors, WSDL→Java, etc.)
   - Custom builders (Eclipse .builders)
   - Classpath magic (.classpath entries not in pom.xml/build.xml)
   ```

2. **Extract build steps to CLI**
   - If Maven: check if `mvn compile` works after adding missing dependencies to pom.xml
   - If Ant: check if `ant compile` works after fixing classpath in build.xml
   - Common fix: copy JARs from Eclipse classpath to `lib/` or add to pom.xml

3. **Test one module first**
   - Don't try to build everything. Pick the smallest module and get CLI build working for that one.

---

## Lv2 → Lv3 Upgrade Path

If stuck at Lv2 (build works, test doesn't):

1. **Identify why tests fail on CLI**
   ```
   Common reasons:
   - Database connection (test needs Oracle, CLI has no Oracle)
     → Fix: add H2 test profile in application-test.properties
   - File paths hardcoded to Windows/Eclipse workspace
     → Fix: use classpath-relative paths
   - Eclipse-specific test runner configuration
     → Fix: ensure JUnit is in test classpath
   ```

2. **Start with unit tests (not integration)**
   - Pure logic tests that don't need DB/network
   - `mvn test -Dtest=*Unit*` or similar filter

3. **Add H2 in-memory DB for integration tests**
   ```xml
   <!-- pom.xml test scope -->
   <dependency>
     <groupId>com.h2database</groupId>
     <artifactId>h2</artifactId>
     <scope>test</scope>
   </dependency>
   ```
   ```properties
   # src/test/resources/application-test.properties
   spring.datasource.url=jdbc:h2:mem:testdb
   spring.datasource.driver-class-name=org.h2.Driver
   spring.jpa.hibernate.ddl-auto=create-drop
   ```
