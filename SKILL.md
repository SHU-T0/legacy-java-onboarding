---
name: legacy-java-onboarding
description: "Onboard Claude Code to legacy Java codebases (Spring Boot, Hibernate, Ant/Maven, CVS/SVN). Three modes: (1) assess — determine environment readiness level Lv0-Lv3 with build time measurement and fallback workflows, (2) setup — generate CLAUDE.md hierarchy, Navigation Map, and .claudeignore with large codebase context management, (3) review — detect legacy anti-patterns and output exact Claude Code prompts to fix them. Also covers: CVS/SVN to local Git workflow, Eclipse-dependent build fallbacks, multi-module CLAUDE.md hierarchy for 100k+ LOC codebases, team coaching strategies. Use when: introducing Claude Code to an existing Java project, analyzing legacy Java architecture, creating CLAUDE.md for Java/Spring/Hibernate projects, assessing CLI build readiness, coaching teams on AI-driven legacy Java development. Triggers: 'legacy Java', 'onboard Claude Code to Java', 'Java CLAUDE.md', 'assess Java project', 'legacy codebase analysis', 'Spring Boot onboarding', 'Hibernate review', 'レガシーJava', 'Java環境構築', /legacy-java-onboarding."
---

# Legacy Java Onboarding for Claude Code

Onboard Claude Code to any legacy Java codebase. Three modes + supporting references.

**Parse $ARGUMENTS**: extract mode (`assess`/`setup`/`review`). Default (no args) = run all three in sequence.

---

## Mode: assess

Determine Claude Code integration level and create an action plan.

### Step 1: Hard Requirements

```bash
git --version && node -v && claude --version
curl -s -o /dev/null -w "%{http_code}" https://api.anthropic.com/v1/messages
# 401 or 405 = reachable. Timeout = blocked.
```

### Step 2: VCS Check

```bash
# Is it already Git?
git status 2>/dev/null && echo "GIT_OK"
# CVS?
ls CVS/Root 2>/dev/null && cat CVS/Root
# SVN?
svn info 2>/dev/null | head -3
```

If CVS/SVN: follow the local Git workflow in [references/vcs-migration.md](references/vcs-migration.md).

### Step 3: Build Chain

```bash
ls pom.xml build.xml build.gradle Makefile .project 2>/dev/null
```

Measure build + test time with detected tool:
```bash
time mvn compile -q 2>&1       # Maven
time ant compile 2>&1           # Ant
time gradle build -q 2>&1      # Gradle
time mvn test 2>&1 | grep -E "Tests run|BUILD"
```

### Step 4: Level Assignment

| Level | Criteria | Capability | Next Action |
|-------|----------|-----------|-------------|
| **Lv3 Full** | CLI build ✓ + CLI test ✓ + Git ✓ | Autonomous modify→build→test→fix | Proceed to `setup` |
| **Lv2 Partial** | CLI build ✓ + Git ✓, test manual | Auto build, human runs tests | See [Lv2 workflow](references/fallback-workflows.md#lv2) |
| **Lv1 Read-only** | Git ✓ only, build needs IDE | Analysis, docs, proposals only | See [Lv1 workflow](references/fallback-workflows.md#lv1) |
| **Lv0 No-Go** | API unreachable | Cannot operate | Check proxy/firewall |

Build time thresholds:
- ≤5 min → Full scope
- 5-15 min → Scope to single module. Add module filter to CLAUDE.md
- \>15 min → Target one module only, restructure build

### Step 5: Spring Boot Version Detection

```bash
# javax = Spring Boot 2.x (pre-Jakarta)
grep -rl "import javax.persistence" --include="*.java" src/ 2>/dev/null | wc -l
# jakarta = Spring Boot 3.x+
grep -rl "import jakarta.persistence" --include="*.java" src/ 2>/dev/null | wc -l
# Explicit version
grep -E "spring-boot.*version|java.version" pom.xml build.gradle 2>/dev/null
```

Output: summary table with level, build tool, build time, test time, Java version, Spring Boot version, Hibernate version, VCS, blocker list.

---

## Mode: setup

Generate CLAUDE.md, Navigation Map, and .claudeignore. For large codebases (100k+ LOC), use hierarchical CLAUDE.md — see [references/large-codebase-strategy.md](references/large-codebase-strategy.md).

### Step 1: Codebase Analysis

Run the analysis script or execute commands manually:

```bash
# Scale
cloc . --quiet 2>/dev/null || find . -name "*.java" | wc -l

# Directory structure
find src/main/java -maxdepth 4 -type d 2>/dev/null | sort

# Package file counts (heaviest = likely core logic)
find src/main/java -name "*.java" | xargs dirname | sort | uniq -c | sort -rn | head -20

# Stack detection
grep -rl "import org.springframework" --include="*.java" src/ 2>/dev/null | wc -l
grep -rl "import.*hibernate\|import javax.persistence\|import jakarta.persistence" --include="*.java" src/ 2>/dev/null | wc -l

# Entry points
grep -rl "@SpringBootApplication\|@Controller\|@RestController\|@WebService" --include="*.java" src/ 2>/dev/null

# Data layer
grep -rl "@Entity\|@Table" --include="*.java" src/ 2>/dev/null

# Service layer
grep -rl "@Service\|@Component" --include="*.java" src/ 2>/dev/null

# Repository layer
grep -rl "@Repository\|extends.*Dao\|extends JpaRepository" --include="*.java" src/ 2>/dev/null

# Tests
find src/test -name "*Test*.java" 2>/dev/null | wc -l
```

### Step 2: Build Navigation Map

Systematic approach (works for unfamiliar codebases):

1. List top packages by file count → these are core areas
2. Find entry points (@Controller, main()) → trace inward
3. Find data layer (@Entity) → trace outward to services
4. Map: Entry Point → Service → Repository → Entity

Create table:
```markdown
| Area | Path | Files | Key Classes | Role |
|------|------|-------|-------------|------|
```

Limit to 15-20 rows. 60% accuracy is fine on Day 1 — refine iteratively.

If project owner is available, show map and ask: "Is this roughly correct? What am I missing?"

### Step 3: Generate CLAUDE.md

Use [references/claude-md-java-template.md](references/claude-md-java-template.md). Fill all `[FILL]` placeholders from analysis data.

For multi-module projects (3+ modules), also create sub-CLAUDE.md files — see [references/large-codebase-strategy.md](references/large-codebase-strategy.md).

### Step 4: Generate .claudeignore

```
target/
build/
bin/
out/
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
node_modules/
.mvn/wrapper/
```

Add project-specific exclusions (large binary dirs, archived code, vendor libs).

---

## Mode: review

Detect legacy patterns and output actionable Claude Code fix prompts. Read [references/legacy-patterns.md](references/legacy-patterns.md) for the full pattern catalog with detailed explanations.

### Detection Commands

Run all:
```bash
echo "=== H1: N+1 EAGER ===" && grep -rn "FetchType.EAGER" --include="*.java" src/
echo "=== H2: Entity in API ===" && grep -rn "ResponseEntity" --include="*.java" src/main/java/**/controller/
echo "=== M1: Field injection ===" && grep -rn "@Autowired" --include="*.java" src/main/
echo "=== M2: java.util.Date ===" && grep -rn "import java.util.Date" --include="*.java" src/
echo "=== M3: Magic numbers ===" && grep -rn "getStatus() ==\|getType() ==\|setStatus(" --include="*.java" src/main/
echo "=== M5: Manual JDBC ===" && grep -rn "DriverManager\|PreparedStatement" --include="*.java" src/
echo "=== L1: javax.persistence ===" && grep -rl "import javax.persistence" --include="*.java" src/ | wc -l
echo "=== L2: Catch-all ===" && grep -rn "catch (Exception\|catch (Throwable" --include="*.java" src/main/
```

For H2 (Entity in API): manually inspect controller return types — check if @Entity objects are returned directly without DTO wrapper.

For H3 (No tests): compare main packages vs test packages:
```bash
diff <(find src/main/java -name "*.java" | xargs dirname | sort -u | sed 's|src/main/java|src/test/java|') <(find src/test/java -name "*.java" | xargs dirname | sort -u) | grep "^<"
```

### Output Format

For each finding:
```
## [Pattern Name] — [Severity]
- **Where**: file.java:line
- **Impact**: [why it matters for this project]
- **Fix prompt**: `"[exact Claude Code prompt to fix this]"`
```

### Summary Table

Output: total findings by severity, estimated fix effort per pattern, and recommended priority order (highest ROI first).

---

## References

| File | When to read |
|------|-------------|
| [claude-md-java-template.md](references/claude-md-java-template.md) | setup mode — CLAUDE.md generation |
| [legacy-patterns.md](references/legacy-patterns.md) | review mode — detailed pattern descriptions and fix prompts |
| [large-codebase-strategy.md](references/large-codebase-strategy.md) | setup mode with 100k+ LOC — CLAUDE.md hierarchy, .claudeignore, subagent strategy |
| [vcs-migration.md](references/vcs-migration.md) | assess mode — CVS/SVN to local Git workflow |
| [fallback-workflows.md](references/fallback-workflows.md) | assess mode — Lv1/Lv2 alternative workflows when CLI build/test unavailable |
