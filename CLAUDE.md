# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

genmmit is an AI-powered Git commit message generator implemented as a Git hook (`prepare-commit-msg`). Single Bash script (~314 lines) calling OpenAI-compatible APIs to generate commit messages from staged diffs.

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

**Template resolution order:**
1. If path contains `/`: expand `~` and use as-is
2. `~/.config/genmmit/templates/<name>`
3. `<script_dir>/templates/<name>` (for non-hook usage)

## Script Flow

1. Check `commit_source` - skip if `message` or `merge`
2. Load configs: global → project `.genmmit` → env vars
3. Validate: `GENMMIT_API_URL`, `GENMMIT_API_KEY`, `GENMMIT_MODEL` required
4. Get `git diff --staged`, truncate to `GENMMIT_DIFF_CHARS` (default 4000, 0=disable)
5. Build prompt: inline `GENMMIT_PROMPT` or template file + diff
6. Call API, parse `choices[0].message.content`
7. Write to commit message file (or stdout if no file)

**Silent failure:** All errors `exit 0` to avoid blocking commits.

## Testing

```bash
# Test script directly (stages changes first)
git add -A && ./genmmit /tmp/test-msg && cat /tmp/test-msg

# Debug with trace
bash -x ./genmmit /tmp/test-msg

# Check logs
cat ~/.config/genmmit/error.log

# Test with different log levels
GENMMIT_LOG_LEVEL=debug ./genmmit /tmp/test-msg
```

## Installation

```bash
# Install to project
cp genmmit .git/hooks/prepare-commit-msg && chmod +x .git/hooks/prepare-commit-msg

# Or install globally (if using core.hooksPath)
cp genmmit ~/.config/git/hooks/prepare-commit-msg && chmod +x ~/.config/git/hooks/prepare-commit-msg
```

## Dependencies

- `curl` - API requests
- `jq` - JSON parsing (used for escaping prompt and parsing response)
