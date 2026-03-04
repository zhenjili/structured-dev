---
name: structured-dev
description: Systematic feature-driven development for complex coding projects. Maintains feature_list.json for tracking and progress.txt for session continuity. Use this skill when the user invokes /structured-dev, when you detect feature_list.json in the project root, or when the user asks for structured/systematic development of a multi-feature project. Also trigger when users mention feature tracking, incremental development, long-running projects, or multi-session coding tasks.
---

# Structured Development

A disciplined, verification-first approach to building complex software projects. You work through features one at a time, tracking progress in persistent artifacts that survive across sessions.

## Mode Detection

Check the project root for these files:

1. **`feature_list.json` exists** → Continue Mode (skip to "Continue: 11-Step Workflow")
2. **Neither file exists** → Initialize Mode (read on)

If `feature_list.json` exists but `progress.txt` doesn't, create the progress file and enter Continue Mode.

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

Don't catalog all existing functionality. Only track what you're changing, plus a baseline check to catch regressions.

#### Gather Requirements

Ask the user what they want to accomplish. If they haven't specified, ask: **"What changes do you want to make to this project?"**

Explore the relevant parts of the codebase that will be affected. Focus on the areas being modified and their immediate dependencies.

#### Generate Focused feature_list.json

Read `references/feature_list_schema.md` for the complete schema.

Feature #1 is always a baseline check (existing tests pass, project builds). Run it immediately — if it passes, mark `true`. Remaining features cover only the requested changes, typically 5-20 features.

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

Write `progress.txt`:

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
git add feature_list.json progress.txt
git commit -m "feat: initialize structured development tracking

- Mode: [greenfield/modification]
- Generated feature_list.json with N features
- Created progress.txt for session continuity"
```

### Begin Work

Transition to Continue Mode and start implementing the first pending feature.

---

## Continue: 11-Step Workflow

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

**Read the user's latest message** — it likely explains what they want instead.

**Decision tree:**

- **User gave corrective feedback** (e.g., "use SQL instead of ORM", "don't change that file"):
  1. Revert the uncommitted changes: `git checkout -- .`
  2. Acknowledge the feedback and explain how you'll approach differently
  3. Proceed to Step 6 (Select Feature) with the user's guidance in mind

- **User wants to skip this feature** (e.g., "skip this one", "move on"):
  1. Revert: `git checkout -- .`
  2. Note the skip in `progress.txt`
  3. Proceed to Step 6 with the next eligible feature

- **Partial work looks good, user wants to continue** (e.g., "keep going", "continue"):
  1. Keep the uncommitted changes
  2. Proceed to Step 7 (Implement) to finish the current feature

- **Unclear what the user wants**:
  1. Show the uncommitted changes summary
  2. Ask: revert and start fresh, or keep and continue?

### Step 2: Understand User Intent

Read the user's message — it determines what you do next.

Classify intent:

| Intent | Signal | Action |
|--------|--------|--------|
| **Add new feature** | "add dark mode", "I also need pagination" | → Go to Step 2a: Add Features |
| **Modify approach** | "use Redis instead", "don't touch that file" | → Note the constraint, proceed to Step 6 (Select Feature) |
| **Continue working** | "continue", "keep going", "next" | → Proceed to Step 3 normally |
| **Fix/redo something** | "that's broken", "redo feature #3" | → Identify the issue, proceed to Step 6 targeting that feature |
| **Skip a feature** | "skip #4", "that's not needed" | → Note skip in progress, proceed to Step 6 |
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
5. Proceed to Step 3 — the new features will be picked up naturally by Step 6

### Step 3: Read Progress

Read `progress.txt` to understand:
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

### Step 6: Select Feature(s)

Identify all **eligible features** — those where `passes` is `false` and all `depends_on` features pass.

**If the user requested a specific feature** in Step 2, prioritize that one. If they asked to redo a feature, target that one.

**If the user gave implementation guidance** (e.g., "use SQLite not Postgres"), note it — their instructions take priority.

Choose from eligible features freely — you don't have to follow ID order. Pick what makes the most sense given context, dependencies, and user intent.

#### Serial vs Parallel

By default, work on **one feature at a time** — it keeps verification clean and git history readable.

But when multiple eligible features are independent (no shared files, no overlapping concerns), you can implement them in parallel using subagents. This is a judgment call based on:
- Do the features touch different parts of the codebase?
- Can each be verified independently?
- Is the speedup worth the coordination overhead?

When parallelizing, each subagent should complete a full cycle (implement → verify → update feature list) for its feature. Coordinate commits to avoid conflicts.

Announce your choice: "Working on Feature #N: [description]" (or "Working on Features #N and #M in parallel").

### Step 7: Implement

Write the code for the selected feature:
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

Without this step, the next session starts blind. Progress notes are the only bridge between sessions — 30 seconds of writing saves minutes of re-discovery.

Append to `progress.txt`:
```
### Feature #N: [description]
- Status: COMPLETE
- Implementation: [brief summary of what was built and key decisions made]
- Verification: [how it was tested, what commands were run]
- Files changed: [key files that were added or modified]
- Notes: [anything the next session should know — gotchas, trade-offs, related work]
```

Then commit the progress update:
```
git add feature_list.json progress.txt
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
3. **Update progress**: Write a session summary to `progress.txt`:

```
## Session Summary
- Features completed this session: [list]
- Features remaining: [count]
- Overall progress: [completed]/[total] ([percentage]%)
- Next session should start with: Feature #[N] - [description]
- Known issues or blockers: [any]
```

4. **Commit progress**: `git add progress.txt && git commit -m "docs: end of session progress update"`
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

**Why focus on few features at a time?** Context switches between half-done features lead to bugs and broken state. Finishing before starting the next keeps the codebase always-working.

**Why verify before implementing?** Regressions compound. If you build feature #5 on top of a broken feature #3, you'll waste time debugging the wrong thing.

**Why commit after every feature?** Small, focused commits make it easy to bisect problems and revert cleanly. A giant commit with 5 features is useless when one of them breaks.

**Why progress notes?** You won't remember what happened 3 sessions ago. Neither will the next agent instance. Written context is the only bridge between sessions — without it, the next session wastes time re-discovering what you already figured out.
