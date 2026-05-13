# skill: graphify

## purpose
Before touching any code, build a dependency map of the codebase — which files depend on what, which are load-bearing god nodes, which clusters belong together, which files to read first for the current task. Writes the result to `.agent-context/graph.md` so future sessions reuse it without rebuilding.

Without this, agents read files randomly and miss critical dependencies.

## when_to_use
- Starting work in an unfamiliar codebase
- Debugging a failure where the cause could be anywhere in the call chain
- Refactoring a module that other code depends on
- Deciding which file to edit when multiple could be relevant

## when_not_to_use
- Codebase is under ~10 files — just read them all
- Task is isolated to one known file with no cross-file dependencies
- .agent-context/graph.md already exists and code hasn't changed significantly

## input
```
file_list: list[string]       # All file paths in the codebase
entry_points: list[string]    # Known main files / routes (detected if empty)
task: string                  # What you're about to do — determines relevance
```

## output
Writes `.agent-context/graph.md` and returns:
```json
{
  "graph": {
    "nodes": [
      {
        "id": "string",
        "type": "entry|core|util|config|test|dead",
        "depends_on": ["string"],
        "depended_on_by": ["string"],
        "risk": "high|medium|low"
      }
    ],
    "god_nodes": ["string"],
    "clusters": [{ "name": "string", "files": ["string"], "purpose": "string" }],
    "task_relevant_files": ["string"],
    "safe_to_ignore": ["string"]
  },
  "summary": "string"
}
```

## instructions

```
Build a dependency map. Do not read file contents yet — work from file names and paths first.

Step 1: Identify entry points (main.py, index.ts, app.py, server.js, cmd/, routes/)
Step 2: Read entry points only. Follow import statements 2 levels deep.
Step 3: Classify each file:
  - entry: program starts here
  - core: imported by 3+ other files (high risk to change)
  - util: helper functions, used by many but doesn't own domain logic
  - config: settings, env, constants
  - test: test files
  - dead: not imported anywhere, not an entry point

Step 4: Identify god nodes — files imported by 3+ others
Step 5: Group files into clusters by directory or implied domain
Step 6: For the current task, identify the 3–7 most relevant files to read first

Write output to .agent-context/graph.md.
Return JSON summary. summary field: 2–3 sentences on codebase shape.
task_relevant_files must be 7 or fewer — force prioritization.
```

## constraints
- Max output tokens: 600
- task_relevant_files: max 7 files
- Build from file names first — read files only when needed to trace imports
- Write to .agent-context/graph.md (create .agent-context/ if it doesn't exist)

## example

**Input:**
```
file_list: ["main.py","app/routes/auth.py","app/routes/billing.py","app/models/user.py","app/models/subscription.py","app/services/stripe_service.py","app/services/email_service.py","app/utils/validators.py","app/config.py","tests/test_billing.py"]
entry_points: ["main.py"]
task: "Debug why subscription cancellation doesn't send confirmation email"
```

**Output:**
```json
{
  "graph": {
    "nodes": [
      { "id": "app/config.py", "type": "config", "depends_on": [], "depended_on_by": ["main.py","app/services/stripe_service.py","app/services/email_service.py"], "risk": "high" },
      { "id": "app/routes/billing.py", "type": "core", "depends_on": ["app/models/subscription.py","app/services/stripe_service.py","app/services/email_service.py"], "depended_on_by": ["main.py"], "risk": "medium" },
      { "id": "app/services/email_service.py", "type": "core", "depends_on": ["app/config.py"], "depended_on_by": ["app/routes/billing.py","app/routes/auth.py"], "risk": "medium" },
      { "id": "app/models/subscription.py", "type": "core", "depends_on": [], "depended_on_by": ["app/routes/billing.py"], "risk": "medium" },
      { "id": "tests/test_billing.py", "type": "test", "depends_on": [], "depended_on_by": [], "risk": "low" }
    ],
    "god_nodes": ["app/config.py"],
    "clusters": [
      { "name": "billing", "files": ["app/routes/billing.py","app/models/subscription.py","app/services/stripe_service.py"], "purpose": "Subscription lifecycle and payment processing" },
      { "name": "notifications", "files": ["app/services/email_service.py"], "purpose": "Transactional email sending" }
    ],
    "task_relevant_files": ["app/routes/billing.py","app/services/email_service.py","app/models/subscription.py","app/config.py"],
    "safe_to_ignore": ["app/routes/auth.py","app/utils/validators.py","tests/test_billing.py"]
  },
  "summary": "Small FastAPI app. config.py is the god node — all services depend on it. Cancellation bug likely in billing.py (does it call email_service?) or email_service.py (is the right function being called?). Read those two first."
}
```

## feedback_log
<!-- date | situation | what went wrong | better behavior -->
