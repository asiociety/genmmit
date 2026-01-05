# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

genmmit is an AI-powered Git commit message generator implemented as a Git hook (`prepare-commit-msg`). It's a single Bash script that calls OpenAI-compatible APIs to generate commit messages based on staged changes.

## Architecture

```
genmmit (main script) → Git hook (prepare-commit-msg)
    ↓
~/.config/genmmit/
├── config              # Global config (API credentials, model, defaults)
├── templates/          # Prompt templates (conventional.txt, simple.txt, angular.txt)
└── error.log           # Log file

.genmmit                # Project-level config override
```

**Config priority:** Environment variables > Project `.genmmit` > Global `~/.config/genmmit/config`

**Template resolution:** `GENMMIT_TEMPLATE` value is prefixed with `~/.config/genmmit/templates/`

## Testing

```bash
# Test the script directly (without git commit)
./genmmit /tmp/test-msg

# Debug with trace
bash -x ./genmmit /tmp/test-msg

# Check logs
cat ~/.config/genmmit/error.log
```

## Installation

```bash
# Install to project
cp genmmit .git/hooks/prepare-commit-msg
chmod +x .git/hooks/prepare-commit-msg

# Or install globally (if using core.hooksPath)
cp genmmit ~/.config/git/hooks/prepare-commit-msg
```

## Dependencies

- `curl` - API requests
- `jq` - JSON parsing
