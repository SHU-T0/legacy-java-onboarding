# legacy-java-onboarding

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill for onboarding legacy Java codebases (Spring Boot, Hibernate, Ant/Maven, CVS/SVN).

## Install

```bash
git clone https://github.com/shu-t0/legacy-java-onboarding.git ~/.claude/skills/legacy-java-onboarding
```

## Usage

The skill triggers automatically when working with legacy Java projects, or invoke manually:

| Mode | Command | What it does |
|------|---------|-------------|
| **assess** | `/legacy-java-onboarding assess` | Determine Claude Code integration level (Lv0-Lv3), measure build time, detect VCS |
| **setup** | `/legacy-java-onboarding setup` | Generate CLAUDE.md hierarchy, Navigation Map, .claudeignore |
| **review** | `/legacy-java-onboarding review` | Detect legacy anti-patterns, output exact Claude Code fix prompts |
| **full** | `/legacy-java-onboarding` | Run all three in sequence |

## Integration Levels

| Level | Criteria | Claude Code Capability |
|-------|----------|----------------------|
| Lv3 Full | CLI build + CLI test + Git | Autonomous modify→build→test→fix loop |
| Lv2 Partial | CLI build + Git, test manual | Modify→build auto, test via human |
| Lv1 Read-only | Git only, build requires IDE | Analysis, documentation, proposals |
| Lv0 No-Go | No API connectivity | Cannot operate |

Each level has a defined workflow. Lv1/Lv2 include upgrade paths to reach Lv3.

## Legacy Patterns Detected

| Severity | Patterns |
|----------|----------|
| High | N+1 EAGER fetch, Entity exposed in API, No tests |
| Medium | Field injection, java.util.Date, Magic numbers, Raw List, Manual JDBC |
| Low | javax.persistence (pre-Jakarta), Catch-all exception |

Each finding includes file:line location and an exact Claude Code prompt to fix it.

## Key Features

- **CVS/SVN support**: Local Git wrapper workflow for non-Git environments
- **Large codebase strategy**: CLAUDE.md hierarchy, .claudeignore, subagent delegation for 100k+ LOC
- **Fallback workflows**: Concrete alternatives when CLI build/test is unavailable
- **Build time awareness**: Threshold-based scoping (≤5min go, 5-15min scope, >15min restructure)
- **Spring Boot version detection**: javax vs jakarta, upgrade path guidance

## Structure

```
legacy-java-onboarding/
├── SKILL.md                                  # Core skill (3 modes)
└── references/
    ├── claude-md-java-template.md            # CLAUDE.md fill-in template
    ├── legacy-patterns.md                    # 10 patterns with fix prompts
    ├── large-codebase-strategy.md            # 100k+ LOC: hierarchy, ignore, subagents
    ├── vcs-migration.md                      # CVS/SVN → local Git workflow
    └── fallback-workflows.md                 # Lv1/Lv2 workflows + upgrade paths
```

## License

MIT
