---
name: legacy-java-onboarding
description: "Onboard Claude Code to legacy Java codebases (Spring Boot, Hibernate, Ant/Maven, CVS/SVN). Three modes: (1) assess — determine environment readiness level Lv0-Lv3 with build time measurement, (2) setup — generate CLAUDE.md, Navigation Map, and .claudeignore for the Java project, (3) review — detect legacy anti-patterns (N+1, raw JDBC, field injection, magic numbers) and output exact Claude Code prompts to fix them. Use when: introducing Claude Code to an existing Java project, analyzing legacy Java architecture, creating CLAUDE.md for Java/Spring/Hibernate projects, assessing CLI build readiness, coaching teams on AI-driven legacy Java development. Triggers: 'legacy Java', 'onboard Claude Code to Java', 'Java CLAUDE.md', 'assess Java project', 'legacy codebase analysis', 'Spring Boot onboarding', 'Hibernate review', 'レガシーJava', 'Java環境構築', /legacy-java-onboarding."
---

# Legacy Java Onboarding for Claude Code

Onboard Claude Code to any legacy Java codebase. Three modes: `assess`, `setup`, `review`.

**Parse $ARGUMENTS**: extract mode (`assess`/`setup`/`review`). Default (no args) = run all three in sequence.

## Mode: assess

Determine the Claude Code integration level for this Java project.

### Step 1: Hard Requirements

Run in order. Stop at first failure:
```bash
git --version && node -v && claude --version
curl -s -o /dev/null -w "%{http_code}" https://api.anthropic.com/v1/messages  # 401=OK
```

### Step 2: Build Chain Detection

```bash
ls pom.xml build.xml build.gradle Makefile .project 2>/dev/null
```

Then measure build time with the detected tool:
```bash
time mvn compile -q 2>&1      # Maven
time ant compile 2>&1          # Ant
time gradle build -q 2>&1     # Gradle
```

And test execution:
```bash
time mvn test 2>&1 | grep -E "Tests run|BUILD"
time ant test 2>&1 | grep -E "Tests run|BUILD"
```

### Step 3: Level Assignment

| Level | Criteria | Claude Code Capability |
|-------|----------|----------------------|
| **Lv3 Full** | CLI build ✓ + CLI test ✓ + Git ✓ | Autonomous modify→build→test→fix loop |
| **Lv2 Partial** | CLI build ✓ + Git ✓, test manual | Modify→build auto, test via human/IDE |
| **Lv1 Read-only** | Git ✓ only, build requires IDE | Analysis, documentation, proposals |
| **Lv0 No-Go** | No API connectivity | Cannot operate |

Build time thresholds:
- ≤5 min → Go
- 5-15 min → Scope to single module
- \>15 min → Target one module only, restructure needed

Output a summary table with: level, build tool, build time, test time, Java version, framework versions, blocker list.

---

## Mode: setup

Generate CLAUDE.md, Navigation Map, and .claudeignore.

### Step 1: Codebase Analysis

```bash
cloc . --quiet 2>/dev/null || find . -name "*.java" | wc -l
find src/main/java -maxdepth 4 -type d 2>/dev/null | sort
find src/main/java -name "*.java" | xargs dirname | sort | uniq -c | sort -rn | head -20
```

Detect stack:
```bash
grep -rl "import org.springframework" --include="*.java" src/ 2>/dev/null | wc -l
grep -rl "import.*hibernate\|import javax.persistence\|import jakarta.persistence" --include="*.java" src/ 2>/dev/null | wc -l
grep -E "source.*1\.[0-9]|java.version|sourceCompatibility" pom.xml build.xml build.gradle 2>/dev/null
```

Find entry points and layers:
```bash
grep -rl "@SpringBootApplication\|@Controller\|@RestController" --include="*.java" src/ 2>/dev/null
grep -rl "@Entity\|@Table" --include="*.java" src/ 2>/dev/null
grep -rl "@Service\|@Component" --include="*.java" src/ 2>/dev/null
grep -rl "@Repository\|extends.*Dao" --include="*.java" src/ 2>/dev/null
find src/test -name "*Test*.java" 2>/dev/null | wc -l
```

### Step 2: Build Navigation Map

From analysis results, create:

```markdown
| Area | Path | Files | Key Classes | Role |
|------|------|-------|-------------|------|
```

Prioritize by file count. Limit to 15-20 rows. Verify with project owner if available.

### Step 3: Generate CLAUDE.md

Use the template in [references/claude-md-java-template.md](references/claude-md-java-template.md). Fill all sections from analysis data.

### Step 4: Generate .claudeignore

```
target/
build/
bin/
*.class
*.jar
*.war
*.ear
.idea/
.settings/
.project
.classpath
logs/
generated-sources/
```

Add project-specific exclusions after reviewing directory structure.

---

## Mode: review

Detect legacy patterns and output actionable Claude Code prompts to fix them.

### Patterns to Check

| # | Pattern | Detection Command | Severity |
|---|---------|------------------|----------|
| 1 | N+1 queries (EAGER fetch) | `grep -rn "FetchType.EAGER" --include="*.java"` | High |
| 2 | Entity exposed in API | Controllers returning @Entity directly (no DTO) | High |
| 3 | No test for module | dirs with 0 test files | High |
| 4 | Field injection | `grep -rn "@Autowired" --include="*.java"` on fields | Medium |
| 5 | `java.util.Date` | `grep -rn "import java.util.Date" --include="*.java"` | Medium |
| 6 | Magic numbers | int status/type codes (0,1,2,9) | Medium |
| 7 | Raw List (no generics) | `grep -n "List [a-z]" --include="*.java"` | Medium |
| 8 | Manual JDBC | `grep -rn "DriverManager\|PreparedStatement" --include="*.java"` | Medium |
| 9 | `javax.persistence` | Indicates Spring Boot 2.x (pre-Jakarta) | Low |
| 10 | Catch-all exception | `catch (Exception e)` everywhere | Low |

### Output Format

For each finding:
```
## [Pattern Name]
- **Where**: file.java:line
- **Impact**: [why it matters]
- **Fix prompt**: `"[exact Claude Code prompt to fix this]"`
```

### Summary

Output a table: total findings by severity (High/Medium/Low), estimated fix effort, and recommended priority order.

---

## References

- [CLAUDE.md template for Java projects](references/claude-md-java-template.md)
- [Legacy patterns catalog with fix prompts](references/legacy-patterns.md)
