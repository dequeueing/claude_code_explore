# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code learning project**. The repository contains:

- `claude-code-docs/` - Local mirror of official Claude Code documentation (70 docs)
- `note.md` - User's personal learning notes and observations
- `README.md` - Project description

## Project Structure

```
/
├── README.md              # Project description
├── note.md                # User's learning notes (read this first!)
├── CLAUDE.md              # This file
└── claude-code-docs/      # Third-party documentation mirror
    ├── docs/              # 70 markdown documentation files
    ├── install.sh         # Installer for /docs command
    └── CLAUDE.md          # Separate guidance for the mirror project
```

## Key Conventions

### When User Asks About Documentation

1. **Check `note.md` first** - User may have already made observations
2. **Read from `claude-code-docs/docs/`** - Official docs are there
3. **Reference `docs_manifest.json`** - For available topics and freshness

### When User Explores Features

Priority order for learning:
1. `quickstart.md` - For basics
2. `best-practices.md` - For effective usage
3. `common-workflows.md` - For real-world patterns
4. `features-overview.md` - For extension system
5. `how-claude-code-works.md` - For architecture understanding

### File Locations

- **Official docs**: `claude-code-docs/docs/<topic>.md`
- **User notes**: `note.md` (may contain user insights)
- **Manifest**: `claude-code-docs/docs/docs_manifest.json` (shows last updated dates)

## Common Tasks

### Searching Documentation
```bash
# Find specific topic
grep -r "topic" claude-code-docs/docs/

# Check manifest for available docs
cat claude-code-docs/docs/docs_manifest.json | jq '.files | keys'
```

### Updating Documentation
The `claude-code-docs/` directory is a git submodule/mirror. Updates are handled via:
- GitHub Actions in the mirror repo
- Manual: `cd claude-code-docs && git pull`

## Important Notes

- **Do not modify files in `claude-code-docs/docs/`** - They are mirrored from upstream
- **User's notes in `note.md` are authoritative** - Reflects user's understanding
- **This is a learning project** - Focus on explanation and exploration, not production code
