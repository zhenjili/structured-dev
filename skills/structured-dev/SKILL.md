---
name: structured-dev
description: Systematic feature-driven development for complex coding projects. Maintains feature_list.json for tracking and claude-progress.txt for session continuity. Use this skill when the user invokes /structured-dev, when you detect feature_list.json in the project root, or when the user asks for structured/systematic development of a multi-feature project. Also trigger when users mention feature tracking, incremental development, long-running projects, or multi-session coding tasks.
---

# Structured Development

A disciplined, verification-first approach to building complex software projects. You work through features one at a time, tracking progress in persistent artifacts that survive across sessions.

## Mode Detection

Check the project root for these files:

1. **`feature_list.json` exists** → Continue Mode (skip to "Continue: 10-Step Workflow")
2. **Neither file exists** → Initialize Mode (read on)

If `feature_list.json` exists but `claude-progress.txt` doesn't, create the progress file and enter Continue Mode.

---

## Initialize: Project Setup

This mode runs once per project. First, understand the project, then determine the right initialization approach.

### Step 1: Understand the Project

Thoroughly explore the codebase:
- Read README, CLAUDE.md, package.json/pubspec.yaml/requirements.txt/Cargo.toml etc.
- Identify the tech stack, frameworks, and build system
- Understand existing code structure and patterns
- Check for existing tests and how they run
- Note any CI/CD configuration

Identify the **project type** (Web App, API, Mobile, CLI, Library, Full-Stack). Read `references/project_type_guides.md` for type-specific guidance on categories and typical feature distribution.

### Step 2: Determine Initialization Approach

Ask the user (or infer from context) which approach fits:

| Approach | When to use | Feature list covers |
|----------|-------------|-------------------|
| **Greenfield** | Building a new project from scratch | Everything the project needs |
| **Modification** | Adding features or making changes to an existing project | Only the changes + baseline checks |

Most real-world usage is **Modification** — an existing project where the user wants to add, fix, or refactor specific things. Default to Modification if the project already has substantial code.

---

### Greenfield Initialization

For new projects being built from scratch.

#### Determine Project Scale

| Scale | Feature Count | Signal |
|-------|--------------|--------|
| Small | 10-30 | Single-purpose tool, script, or simple app |
| Medium | 30-80 | Multi-page app, API with several resources, typical SaaS |
| Large | 80-200 | Platform, complex full-stack app, enterprise system |

When in doubt, start smaller. Features can be added later but shouldn't be removed.

#### Generate feature_list.json

Read `references/feature_list_schema.md` for the complete schema. Key principles:

- **Every feature must be independently verifiable** - include concrete test steps
- **Order features by dependency** - lower IDs should be implementable first
- **Use `depends_on` sparingly** - only for true hard dependencies
- **Categories**: setup, core, ui, api, data, integration, testing, polish
- **Each feature = one focused capability** - if it takes more than ~30 minutes of coding, break it down

The `project` section must include `test_command` and `build_command` so future sessions know how to verify work.

Skip to "Create Progress File" below.

---

### Modification Initialization

For existing projects where the user wants to add, fix, or refactor specific things. This is the most common scenario.

The key insight: don't catalog all existing functionality. Only track what you're changing, plus a few baseline checks to catch regressions.

#### Gather Requirements

Ask the user what they want to accomplish. They might say:
- "Add user authentication to this API"
- "Refactor the database layer to use an ORM"
- "Fix the search feature and add pagination"
- "Add a settings page with dark mode support"

If the user hasn't specified, ask: **"What changes do you want to make to this project?"**

Explore the relevant parts of the codebase that will be affected by these changes. You don't need to understand the entire project — focus on the areas being modified and their immediate dependencies.

#### Generate Focused feature_list.json

Read `references/feature_list_schema.md` for the complete schema. The feature list for modifications has a specific structure:

**Feature #1 is always a baseline check:**
```json
{
  "id": 1,
  "category": "testing",
  "description": "Baseline: existing tests pass and project builds successfully",
  "steps": [
    "Run test suite: [test_command] — all existing tests pass",
    "Run build: [build_command] — builds without errors"
  ],
  "depends_on": [],
  "priority": "high",
  "passes": false
}
```

Run the baseline immediately during initialization. If it passes, mark it `true`. This establishes the regression safety net.

**Remaining features cover only the requested changes.** Break the user's requirements into granular, testable features. Typically 5-20 features for a modification effort.

Example — user says "add user authentication to this API":
```json
[
  { "id": 1, "description": "Baseline: existing tests pass and project builds", "passes": true },
  { "id": 2, "description": "User model with email, hashed password, and timestamps", "..." },
  { "id": 3, "description": "POST /auth/register creates a new user account", "..." },
  { "id": 4, "description": "POST /auth/login returns JWT token for valid credentials", "..." },
  { "id": 5, "description": "Auth middleware rejects requests without valid token", "..." },
  { "id": 6, "description": "Protected endpoints return 401 for unauthenticated requests", "..." },
  { "id": 7, "description": "Unit tests cover all auth endpoints and edge cases", "..." }
]
```

Notice: no features for existing endpoints, models, or functionality that isn't being changed. Keep it focused.

#### Present to User for Confirmation

Before writing `feature_list.json`, show the user your proposed feature list as a numbered summary:

```
I'll track these features for this modification:
1. [Baseline] Existing tests pass and project builds ✓ (verified)
2. User model with email, hashed password, and timestamps
3. POST /auth/register creates a new user account
4. ...

Does this look right? Want to add, remove, or adjust anything?
```

After user confirms (or adjusts), write the file.

---

### Create Progress File

Write `claude-progress.txt`:

```
# Progress Notes - [Project Name]

## Session 1 - [Date]
- Initialized structured development tracking
- Mode: [Greenfield / Modification]
- Generated feature_list.json with [N] features across [M] categories
- Project type: [type]
- Goal: [brief description of what we're building/changing]

### Current State
- Baseline verified: [yes/no]
- Features pending: [count]
- Next: Start with feature #[N] ([description])

### Environment
- Test command: [command]
- Build command: [command]
- Key dependencies: [list]
```

### Initial Commit

Stage and commit both files:
```
git add feature_list.json claude-progress.txt
git commit -m "feat: initialize structured development tracking

- Mode: [greenfield/modification]
- Generated feature_list.json with N features
- Created claude-progress.txt for session continuity"
```

### Begin Work

Transition to Continue Mode and start implementing the first pending feature.

---

## Continue: 10-Step Workflow

This is your core loop. Execute these steps in order for each feature you implement.

### Step 1: Orient and Detect Interruption

Get your bearings in the project:
```bash
pwd
ls -la
git log --oneline -10
git status
git diff --stat
```

**Check for interrupted work.** If `git status` shows uncommitted changes (modified/added/deleted files), a previous session or attempt was likely interrupted mid-implementation. This needs to be resolved before proceeding.

**If the working tree is clean** — no interruption, proceed to Step 2.

**If the working tree is dirty** — enter Recovery (see below).

#### Recovery: Handling Interrupted Work

Uncommitted changes mean the previous attempt was stopped before completing. The user may have interrupted because:
- They didn't like the approach and want a different one
- Claude was going in the wrong direction
- They want to adjust requirements
- It was an accidental interruption

**Read the user's latest message carefully** — it likely explains what went wrong or what they want instead.

Then assess the state of the uncommitted changes:
1. Run `git diff --stat` to see what files were changed
2. Run the test/build commands to check if the codebase is broken

**Decision tree:**

- **User gave corrective feedback** (e.g., "use SQL instead of ORM", "don't change that file"):
  1. Revert the uncommitted changes: `git checkout -- .`
  2. Acknowledge the feedback and explain how you'll approach differently
  3. Proceed to Step 6 (Select Feature) with the user's guidance in mind

- **User wants to skip this feature** (e.g., "skip this one", "move on"):
  1. Revert: `git checkout -- .`
  2. Note the skip in `claude-progress.txt`
  3. Proceed to Step 6 with the next eligible feature

- **Partial work looks good, user wants to continue** (e.g., "keep going", "continue"):
  1. Keep the uncommitted changes
  2. Proceed to Step 7 (Implement) to finish the current feature

- **Unclear what the user wants**:
  1. Show them the uncommitted changes summary
  2. Ask: "I see uncommitted changes from a previous attempt. Should I revert them and start fresh, or keep them and continue?"

After recovery, always verify the codebase is in a clean building/testing state before implementing new code.

### Step 2: Understand User Intent

**This step is critical.** Every time you enter Continue Mode, the user has sent a message. Read it carefully before doing anything mechanical. The user's message determines what you do next.

Classify the user's intent:

| Intent | Signal | Action |
|--------|--------|--------|
| **Add new feature** | "add dark mode", "I also need pagination" | → Go to Step 2a: Add Features |
| **Modify approach** | "use Redis instead", "don't touch that file" | → Note the constraint, proceed to Step 5 |
| **Continue working** | "continue", "keep going", "next" | → Proceed to Step 3 normally |
| **Fix/redo something** | "that's broken", "redo feature #3" | → Identify the issue, proceed to Step 5 targeting that feature |
| **Skip a feature** | "skip #4", "that's not needed" | → Note skip in progress, proceed to Step 5 |
| **Status check** | "what's the progress?", "where are we?" | → Report status from feature list + progress file, then ask what to do next |
| **Just invoked /structured-dev** | No specific request | → Proceed to Step 3 normally |

#### Step 2a: Add New Features

When the user requests functionality not in the feature list:

1. Break their request into granular, testable features
2. Present the proposed features for confirmation:
   ```
   I'll add these features to the tracking list:
   #[next_id]. [description]
   #[next_id+1]. [description]
   ...
   Look good?
   ```
3. After confirmation, add to `feature_list.json`:
   - Use next sequential IDs
   - Set appropriate category, steps, depends_on
   - Update `metadata.total_features`
4. Commit the updated feature list:
   ```
   git add feature_list.json
   git commit -m "feat: add features #X-#Y to tracking list"
   ```
5. Proceed to Step 3 — the new features will be picked up naturally by Step 5

### Step 3: Read Progress

Read `claude-progress.txt` to understand:
- What was accomplished in previous sessions
- What feature was being worked on or is next
- Any noted blockers or decisions
- Environment setup needed (servers, databases, etc.)

### Step 4: Read Feature List

Read `feature_list.json` and assess current state:
- Count completed (`passes: true`) vs remaining features
- Identify the next eligible feature (all `depends_on` satisfied)
- Note the test/build commands from the `project` section

### Step 5: Verify Existing Work

Before touching anything new, confirm existing work still passes:

1. Run the project's test command
2. Run the build command
3. If any previously-passing feature now fails, **fix it first** before moving on

This step is critical. Regressions caught early are cheap to fix. Never skip verification.

### Step 6: Select One Feature

Choose the next feature to implement:
- If the user requested a specific feature in Step 2, implement that one
- If the user asked to redo a feature, select that feature (its `passes` should still be `false`, or the issue is a regression — fix it)
- Otherwise, pick the lowest-ID feature where `passes` is `false` and all `depends_on` features pass
- **If the user gave implementation guidance** (e.g., "use SQLite not Postgres"), note it — their instructions take priority over your default approach
- Announce your choice: "Working on Feature #N: [description]"

### Step 7: Implement

Write the code for this single feature:
- Follow existing code patterns and conventions
- Keep changes focused - don't refactor unrelated code
- Write tests alongside implementation if the project has a test framework
- If the feature is more complex than expected, implement a minimal working version first

### Step 8: Verify

Run the feature's verification steps from the `steps` array:
- Execute each step and confirm expected behavior
- Run the full test suite to check for regressions
- If verification fails, debug and fix before proceeding

### Step 9: Update Feature List

Update `feature_list.json`:
- Set `passes: true` for the completed feature
- Update `metadata.completed_features` count
- Update `metadata.last_updated` timestamp
- **Never modify any other field** - descriptions, steps, and IDs are immutable

### Step 10: Commit

Create a focused git commit:
```
git add -A
git commit -m "feat: implement feature #N - [short description]

[Brief summary of what was implemented and how it was verified]"
```

Use conventional commit prefixes: `feat:`, `fix:`, `test:`, `refactor:`, `docs:`

### Step 11: Update Progress

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
- If user wants to continue, go back to Step 6 with the next feature
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

### User Wants to Skip or Reorder

These are handled in Step 2 (Understand User Intent), but for clarity:
- **Skip**: Leave the feature as `passes: false`, move to the next eligible one. Note the skip in progress.
- **Reorder**: Pick the requested feature if its dependencies are met. If not, explain which dependencies need to be completed first.
- **Add new features**: Handled in Step 2a — break into granular features, confirm with user, add to list, commit.

### Tests Don't Exist Yet

If the project has no test framework:
1. The first "setup" category feature should establish the test framework
2. Until then, verification steps should be manual checks you can perform
3. Describe what you checked in the verification notes

### Project Already Has Code

This is the default case — use Modification Initialization. Only track the changes you're making, not all existing functionality. Feature #1 (baseline check) serves as the regression safety net for everything that already works.

---

## Principles

These aren't arbitrary rules - they exist because long-running development across sessions is fragile without discipline.

**Why immutable features?** Changing descriptions mid-project creates confusion about what was actually verified. If requirements change, add new features instead.

**Why one feature at a time?** Context switches between half-done features lead to bugs and broken state. Finishing one thing completely before starting the next keeps the codebase always-working.

**Why verify before implementing?** Regressions compound. If you build feature #5 on top of a broken feature #3, you'll waste time debugging the wrong thing.

**Why commit after every feature?** Small, focused commits make it easy to bisect problems and revert cleanly. A giant commit with 5 features is useless when one of them breaks.

**Why progress notes?** You won't remember what happened 3 sessions ago. Neither will the next Claude instance. Written context is the bridge between sessions.
