# CVS/SVN to Local Git Workflow

Claude Code requires Git. If the project uses CVS or SVN, create a local Git wrapper.

## CVS → Local Git

```bash
# 1. Checkout from CVS (or use existing working copy)
cvs -d :pserver:user@server:/cvsroot checkout module
cd module

# 2. Initialize local Git
git init

# 3. Exclude CVS metadata
echo "CVS/" >> .gitignore
echo "*.class" >> .gitignore
echo "target/" >> .gitignore
echo "build/" >> .gitignore

# 4. Initial commit
git add -A
git commit -m "Initial import from CVS"
```

### Daily Workflow

```
CVS Repository (source of truth)
  │
  ├── git init (local Git on top)
  │   ├── Claude Code works here
  │   ├── git add / git commit (local)
  │   └── Changes tracked in local Git
  │
  └── Human reviews changes, then:
      cvs commit -m "description"  (pushes to CVS)
```

### Syncing CVS → Git

When others commit to CVS:
```bash
cvs update          # Pull CVS changes
git add -A          # Stage CVS changes in Git
git commit -m "Sync from CVS"
```

### Syncing Git → CVS

After Claude Code makes changes:
```bash
# Review changes
git diff HEAD~1

# If approved, commit to CVS
cvs commit -m "Changes made with Claude Code assistance"
```

## SVN → Local Git

```bash
# Option 1: git-svn (maintains history link)
git svn clone svn://server/repo/trunk local-repo
cd local-repo
# git svn rebase  ← pull SVN changes
# git svn dcommit ← push to SVN

# Option 2: Simple export (no history link)
svn checkout svn://server/repo/trunk local-repo
cd local-repo
git init && git add -A && git commit -m "Initial import from SVN"
```

git-svn is preferred over simple export because it maintains bidirectional sync.

## Full CVS → Git Migration (when ready)

For permanent migration (Phase 2+), use cvs2git:

```bash
# Install
pip install cvs2svn  # or: apt-get install cvs2svn

# Convert (with Japanese encoding support)
cvs2git --blobfile=git-blob.dat --dumpfile=git-dump.dat \
        --encoding=euc-jp --fallback-encoding=utf-8 \
        /path/to/cvs/repo/module

# Import into new Git repo
mkdir new-repo && cd new-repo
git init
cat ../git-blob.dat ../git-dump.dat | git fast-import
git checkout main
```

**Important for Japanese projects**: specify `--encoding=euc-jp` or `--encoding=shift_jis` to prevent commit message corruption.
