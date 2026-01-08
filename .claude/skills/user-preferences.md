# User Preferences and Guidelines

## Documentation Philosophy

**DO NOT create extraneous documentation files.** The user strongly dislikes AI-generated summary and documentation files such as:
- SUMMARY.md
- BUGFIX.md
- CHANGELOG.md
- IMPLEMENTATION_NOTES.md
- Any other AI-generated meta-documentation

**Exception:** Only create documentation files when explicitly requested by the user.

**Rationale:** The code, comments, and existing documentation (README.md, architecture.md) should be sufficient. Additional AI-generated markdown files create clutter and rarely add value.

## Emoji Usage

**DO NOT use emojis** in:
- Log messages
- Code comments
- Console output
- Documentation
- Pretty much anywhere

**Rationale:** Emojis are unprofessional and add visual noise. Text-based indicators (like "ERROR:", "SUCCESS:", etc.) are clearer and more appropriate for technical projects.

## Startup Scripts

**DO NOT create startup scripts** such as:
- run_tests.bat
- run_tests.sh
- start.sh
- Any wrapper scripts for simple CLI commands

**Rationale:** Manual configuration in a CLI is easier and more flexible. Developers working on this project can handle typing commands directly. Wrapper scripts add an unnecessary layer of abstraction and maintenance burden.

**Preferred approach:** Document the actual commands in README.md or inline comments, not in separate scripts.

## Summary

When working on this project:
1. Fix bugs, write code, create tests - but skip the AI-generated summary documents
2. Use clear text indicators instead of emojis
3. Let users run their own CLI commands instead of creating wrapper scripts
4. Focus on the actual work, not meta-documentation about the work
