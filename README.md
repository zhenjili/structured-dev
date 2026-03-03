# structured-dev

A Claude Code skill for systematic, feature-driven development of complex projects.

## What it does

When you invoke `/structured-dev`, Claude adopts a disciplined development workflow:

1. **Initialization**: Analyzes the project, generates a `feature_list.json` with granular testable features, and creates `claude-progress.txt` for session logging
2. **Implementation**: Works through features one at a time - implement, verify, commit, repeat
3. **Continuation**: On new sessions, reads progress artifacts to seamlessly resume work

## Key concepts

- **feature_list.json**: Sacred tracking file. Features are immutable once created - only the `passes` field changes from `false` to `true`
- **claude-progress.txt**: Session log documenting what was accomplished, what's next, and any blockers
- **Verification-first**: Before starting new work, verify all previously passing features still pass
- **Git discipline**: Commit after each completed feature with descriptive messages
- **Single feature focus**: One feature at a time, done right

## Inspired by

- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/long-running-agents) by Anthropic
- [autonomous-coding](https://github.com/anthropics/autonomous-coding) reference implementation

## Usage

```
/structured-dev
```

Or simply start working on a project that already has `feature_list.json` - the skill will detect it and activate.
