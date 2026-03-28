# Large Codebase Strategy (100k+ LOC)

## The Problem

Claude Code's context window is large (1M tokens) but not infinite. A 500k LOC Java monolith cannot be fully loaded. Without strategy, Claude Code grep-searches blindly and produces unfocused results.

## Strategy 1: CLAUDE.md Hierarchy

For multi-module projects, place CLAUDE.md at each level:

```
project-root/
├── CLAUDE.md                    # Global: project overview, shared conventions, Navigation Map
├── .claudeignore                # Global exclusions
├── module-core/
│   └── CLAUDE.md                # Module-specific: build commands, local patterns, key classes
├── module-web/
│   └── CLAUDE.md                # Module-specific
├── module-batch/
│   └── CLAUDE.md                # Module-specific
└── .claude/
    └── rules/
        ├── spring-conventions.md    # Loaded on demand (not always in context)
        ├── hibernate-patterns.md
        └── naming-conventions.md
```

**Root CLAUDE.md** contains:
- Project-wide architecture overview
- Navigation Map (which module does what)
- Shared conventions and guardrails
- Global build/test commands

**Module CLAUDE.md** contains:
- Module-specific build commands (if different from root)
- Key classes in this module
- Module-specific guardrails
- Dependencies on other modules

**`.claude/rules/`** contains:
- Framework-specific rules (loaded only when Claude determines they're needed)
- Keeps root CLAUDE.md lean

## Strategy 2: .claudeignore for Context Efficiency

Aggressively exclude directories Claude doesn't need:

```
# Build output
target/
build/
bin/
out/

# Binary artifacts
*.class
*.jar
*.war
*.ear

# IDE
.idea/
.settings/
*.iml

# Large/irrelevant directories
docs/legacy/
archived-modules/
vendor/
generated-sources/
**/test-data/large-files/

# Dependencies
node_modules/
.mvn/wrapper/
```

Rule of thumb: if Claude won't need to read or modify it, exclude it.

## Strategy 3: Subagent Delegation

For large investigations, delegate to subagents to keep main context clean:

```
Main session: "Analyze the dependency graph of module-core"
  → Subagent runs grep/find across the entire codebase
  → Returns summary: "module-core depends on module-common (15 classes) and module-data (8 classes)"
  → Main session stays focused
```

In CLAUDE.md, instruct:
```markdown
## Context Management
- For codebase-wide searches, delegate to a subagent
- For dependency analysis across modules, use subagent
- Keep main session focused on the current task/module
```

## Strategy 4: Module Scoping

When build time exceeds 5 minutes, scope Claude Code to one module:

```markdown
## Current Scope
This session focuses on `module-web/` only.
Do NOT modify files outside this module without explicit approval.

## Module Build
- Build this module: `mvn compile -pl module-web -am`
- Test this module: `mvn test -pl module-web`
```

`-pl module-web -am` = build only module-web and its dependencies (not the entire project).

## Strategy 5: Navigation Map for Large Projects

For 50+ packages, organize the Navigation Map by layer, not alphabetically:

```markdown
## Navigation Map

### API Layer (inbound)
| Package | Key Classes | Responsibility |
|---------|-------------|---------------|
| controller.account | AccountController | Account REST API |
| controller.customer | CustomerController | Customer REST API |

### Business Logic
| Package | Key Classes | Responsibility |
|---------|-------------|---------------|
| service.account | AccountService, TransferService | Account operations |
| service.customer | CustomerService | Customer CRUD |

### Data Access
| Package | Key Classes | Responsibility |
|---------|-------------|---------------|
| repository | AccountRepo, CustomerRepo | JPA repositories |
| entity | Account, Customer, Transaction | JPA entities |

### Infrastructure
| Package | Key Classes | Responsibility |
|---------|-------------|---------------|
| config | SecurityConfig, DBConfig | Spring configuration |
| util | DateUtil, StringUtil | Shared utilities |
```
