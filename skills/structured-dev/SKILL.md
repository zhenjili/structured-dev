---
name: structured-dev
description: Systematic feature-driven development for complex coding projects. Maintains feature_list.json for tracking and claude-progress.txt for session continuity. Use this skill when the user invokes /structured-dev, when you detect feature_list.json in the project root, or when the user asks for structured/systematic development of a multi-feature project. Also trigger when users mention feature tracking, incremental development, long-running projects, or multi-session coding tasks.
---

# Structured Development

A disciplined, verification-first approach to building complex software projects. You work through features one at a time, tracking progress in persistent artifacts that survive across sessions.

## Mode Detection

Check the project root for these files:

1. **`feature_list.json` exists** → Continue Mode (skip to "Continue: 10-Step Workflow")
2. **Neither file exists** → Initialize Mode (start with "Initialize: Project Setup")

If `feature_list.json` exists but `claude-progress.txt` doesn't, create the progress file and enter Continue Mode.

---

## Initialize: Project Setup

This mode runs once per project. Your goal: understand the project deeply, then generate a comprehensive feature list that will guide all future development.

### Step 1: Understand the Project

Thoroughly explore the codebase:
- Read README, CLAUDE.md, package.json/pubspec.yaml/requirements.txt/Cargo.toml etc.
- Identify the tech stack, frameworks, and build system
- Understand existing code structure and patterns
- Check for existing tests and how they run
- Note any CI/CD configuration

Identify the **project type** (Web App, API, Mobile, CLI, Library, Full-Stack). Read `references/project_type_guides.md` for type-specific guidance on categories and typical feature distribution.

### Step 2: Determine Project Scale

Assess the project's scope to calibrate feature granularity:

| Scale | Feature Count | Signal |
|-------|--------------|--------|
| Small | 10-30 | Single-purpose tool, script, or simple app |
| Medium | 30-80 | Multi-page app, API with several resources, typical SaaS |
| Large | 80-200 | Platform, complex full-stack app, enterprise system |

When in doubt, start smaller. Features can be added later but shouldn't be removed.

### Step 3: Generate feature_list.json

Read `references/feature_list_schema.md` for the complete schema. Key principles:

- **Every feature must be independently verifiable** - include concrete test steps
- **Order features by dependency** - lower IDs should be implementable first
- **Use `depends_on` sparingly** - only for true hard dependencies
- **Categories**: setup, core, ui, api, data, integration, testing, polish
- **Each feature = one focused capability** - if it takes more than ~30 minutes of coding, break it down

The `project` section must include `test_command` and `build_command` so future sessions know how to verify work.

Write the file to the project root as `feature_list.json`.

### Step 4: Create claude-progress.txt

Write the initial progress file:

```
# Progress Notes - [Project Name]

## Session 1 - [Date]
- Initialized structured development tracking
- Generated feature_list.json with [N] features across [M] categories
- Project type: [type], Scale: [scale]

### Current State
- All features pending
- Next: Start with feature #1 ([description])

### Environment
- Test command: [command]
- Build command: [command]
- Key dependencies: [list]
```

### Step 5: Initial Commit

Stage and commit both files:
```
git add feature_list.json claude-progress.txt
git commit -m "feat: initialize structured development tracking

- Generated feature_list.json with N features
- Created claude-progress.txt for session continuity"
```

### Step 6: Begin Work

Transition to Continue Mode and start implementing the first feature.

---

## Continue: 10-Step Workflow

This is your core loop. Execute these steps in order for each feature you implement.

### Step 1: Orient

Get your bearings in the project:
```bash
pwd
ls -la
git log --oneline -10
git status
```

### Step 2: Read Progress

Read `claude-progress.txt` to understand:
- What was accomplished in previous sessions
- What feature was being worked on or is next
- Any noted blockers or decisions
- Environment setup needed (servers, databases, etc.)

### Step 3: Read Feature List

Read `feature_list.json` and assess current state:
- Count completed (`passes: true`) vs remaining features
- Identify the next eligible feature (all `depends_on` satisfied)
- Note the test/build commands from the `project` section

### Step 4: Verify Existing Work

Before touching anything new, confirm existing work still passes:

1. Run the project's test command
2. Run the build command
3. If any previously-passing feature now fails, **fix it first** before moving on

This step is critical. Regressions caught early are cheap to fix. Never skip verification.

### Step 5: Select One Feature

Choose the next feature to implement:
- Pick the lowest-ID feature where `passes` is `false` and all `depends_on` features pass
- If user has requested a specific feature, prioritize that (but warn if dependencies aren't met)
- Announce your choice: "Working on Feature #N: [description]"

### Step 6: Implement

Write the code for this single feature:
- Follow existing code patterns and conventions
- Keep changes focused - don't refactor unrelated code
- Write tests alongside implementation if the project has a test framework
- If the feature is more complex than expected, implement a minimal working version first

### Step 7: Verify

Run the feature's verification steps from the `steps` array:
- Execute each step and confirm expected behavior
- Run the full test suite to check for regressions
- If verification fails, debug and fix before proceeding

### Step 8: Update Feature List

Update `feature_list.json`:
- Set `passes: true` for the completed feature
- Update `metadata.completed_features` count
- Update `metadata.last_updated` timestamp
- **Never modify any other field** - descriptions, steps, and IDs are immutable

### Step 9: Commit

Create a focused git commit:
```
git add -A
git commit -m "feat: implement feature #N - [short description]

[Brief summary of what was implemented and how it was verified]"
```

Use conventional commit prefixes: `feat:`, `fix:`, `test:`, `refactor:`, `docs:`

### Step 10: Update Progress

Append to `claude-progress.txt`:
```
### Feature #N: [description]
- Status: COMPLETE
- Implementation: [brief summary]
- Verification: [how it was tested]
- Files changed: [key files]
```

Then commit the progress update:
```
git add feature_list.json claude-progress.txt
git commit -m "docs: update progress after completing feature #N"
```

### Loop or End

After completing a feature:
- If context window is getting full (you feel the conversation is very long), proceed to Session End
- If user wants to continue, go back to Step 5 with the next feature
- If all features pass, celebrate and proceed to Session End

---

## Session End Checklist

Before ending any session, always:

1. **Verify clean state**: Run tests, confirm no regressions
2. **Commit all work**: No uncommitted changes should remain
3. **Update progress**: Write a session summary to `claude-progress.txt`:

```
## Session Summary
- Features completed this session: [list]
- Features remaining: [count]
- Overall progress: [completed]/[total] ([percentage]%)
- Next session should start with: Feature #[N] - [description]
- Known issues or blockers: [any]
```

4. **Commit progress**: `git add claude-progress.txt && git commit -m "docs: end of session progress update"`
5. **Never leave broken state**: If a feature is half-done and failing, either finish it or revert

---

## Edge Cases

### Context Window Getting Full

If you sense the conversation is getting very long:
1. Complete the current feature if possible (or revert if broken)
2. Write detailed progress notes about where you stopped
3. Note any in-progress decisions or partial work
4. The next session will pick up from your notes

### Feature Too Complex

If a feature turns out to be significantly more complex than anticipated:
1. Implement the simplest working version
2. Mark the feature as passing if the core behavior works
3. Note in progress what was simplified and what could be enhanced
4. Suggest the user add new features for the enhanced version

### User Wants to Add Features

When the user requests functionality not in the feature list:
1. Add new features at the end of the list (next available ID)
2. Use appropriate category and include verification steps
3. Set appropriate `depends_on` if they depend on existing features
4. Update `metadata.total_features`
5. Commit the updated feature list

### User Wants to Skip or Reorder

- **Skip**: Leave the feature as `passes: false`, move to the next eligible one. Note the skip in progress.
- **Reorder**: Pick the requested feature if its dependencies are met. If not, explain which dependencies need to be completed first.

### Tests Don't Exist Yet

If the project has no test framework:
1. The first "setup" category feature should establish the test framework
2. Until then, verification steps should be manual checks you can perform
3. Describe what you checked in the verification notes

### Project Already Has Code

This is fine and expected. The feature list should cover both:
- Existing functionality (mark as `passes: true` after verifying)
- New functionality to be built (mark as `passes: false`)

Run verification on existing features during initialization.

---

## Principles

These aren't arbitrary rules - they exist because long-running development across sessions is fragile without discipline.

**Why immutable features?** Changing descriptions mid-project creates confusion about what was actually verified. If requirements change, add new features instead.

**Why one feature at a time?** Context switches between half-done features lead to bugs and broken state. Finishing one thing completely before starting the next keeps the codebase always-working.

**Why verify before implementing?** Regressions compound. If you build feature #5 on top of a broken feature #3, you'll waste time debugging the wrong thing.

**Why commit after every feature?** Small, focused commits make it easy to bisect problems and revert cleanly. A giant commit with 5 features is useless when one of them breaks.

**Why progress notes?** You won't remember what happened 3 sessions ago. Neither will the next Claude instance. Written context is the bridge between sessions.
