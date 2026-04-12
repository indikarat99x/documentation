---
title: Rules Configuration
description: How rules.json controls what the agent does when a webhook arrives.
---

`rules.json` is the single configuration surface that controls **what the agent does** when a webhook arrives. Each entry in the JSON array is a self-contained rule set that maps a webhook name to one or more **execution blocks**. Each block independently declares payload filters, input extraction, plugin installation, and a Claude Code prompt template — so a single inbound event can fan out into multiple specialised workflows without any custom code.

If **multiple** execution blocks in the same rule set match one webhook payload, **each** match is scheduled as its own run (separate activation / executor session) with that block's inputs, plugins, and prompt.

```
rules.json  →  WebhookRulesEvaluator  →  EventOrchestrator  →  ProcessingWorkflow  →  Executor Container
```

In the **the-agent** reference implementation, the default file is `Knowledge/rules.json`, embedded at agent registration as Xians knowledge document **`Rules`**.

---

## File Structure

`rules.json` is a JSON array of **rule set** objects. Each rule set targets one **webhook** name (case-insensitive) and contains an **executions** array. Each execution is an independent pipeline: optional filters, inputs, plugins, and prompt.

```jsonc
[
  {
    "webhook": "...",
    "executions": [
      {
        "name": "...",
        "match-any": [ ... ],
        "use-inputs": [ ... ],
        "use-plugins": [ ... ],
        "execute-prompt": "..."
      }
    ]
  }
]
```

| Field | Description |
|-------|-------------|
| `webhook` | Webhook name from Xians Agent Studio (must match incoming events) |
| `executions` | One or more execution blocks; optional per-block `name` for logs and skip messages |

Each execution block that passes its `match-any` filters is scheduled independently when multiple blocks match the same payload.

### Evaluation Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│  Incoming Webhook                                                    │
│  name: "Default"   payload: { "action": "opened", ... }              │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │  Find rule set where  │
                    │  webhook matches      │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │  For each execution:  │
                    │  Evaluate match-any   │──── No match? → skip block
                    │  (OR across entries)  │
                    └───────────┬───────────┘
                                │ At least one match-any passes
                    ┌───────────▼───────────┐
                    │  Extract use-inputs   │
                    │  from payload         │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │  Interpolate          │
                    │  execute-prompt       │
                    │  with {{input-name}}  │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │  Start executor with  │
                    │  plugins + prompt     │
                    └───────────────────────┘
```

---

## 1. `webhook`

Case-insensitive match against the webhook name configured in Xians Agent Studio.

```json
"webhook": "Default"
```

Only one rule set per webhook name is used — the **first** matching entry in the `rules.json` array wins.

---

## 2. `match-any` — Payload Filtering

Inside each execution block, `match-any` is an array of filter rules evaluated with **OR logic**: the block passes if **any** entry matches. If `match-any` is omitted or empty, the block passes unconditionally.

```json
"match-any": [
  { "name": "pr-opened-event",       "rule": "action==opened" },
  { "name": "pr-synchronize-event",  "rule": "action==synchronize" }
]
```

| Field  | Description |
|--------|-------------|
| `name` | Human-readable label (for logging and skip reasons) |
| `rule` | A filter expression — see syntax below |

### Filter Expression Syntax

Each rule is a comparison of a **JSON path** against a **literal value**, optionally combined with `&&` (AND) and `||` (OR) operators:

```
<json-path> <operator> <expected-value>
```

Six operators are supported. All string comparisons are case-sensitive and ordinal.

| Operator | Meaning                                         | Missing path returns |
|----------|-------------------------------------------------|----------------------|
| `==`     | Equals                                          | `false`              |
| `!=`     | Not equals                                      | `true`               |
| `^=`     | Starts with (string prefix match)               | `false`              |
| `!^=`    | Does not start with                             | `true`               |
| `*=`     | Contains (substring match)                      | `false`              |
| `!*=`    | Does not contain                                | `true`               |

`^=`, `!^=`, `*=`, and `!*=` only match **string** values — they never match numbers, booleans, or `null`.

### Compound Expressions

Multiple conditions can be combined in a single rule using `&&` (AND) and `||` (OR):

| Operator | Meaning | Precedence |
|----------|---------|------------|
| `&&`     | AND — all conditions in the group must be true | Higher |
| `||`     | OR — at least one group must be true           | Lower  |

`||` has lower precedence than `&&`. The rule is split into OR-groups first, then each group is split into AND-conditions.

```jsonc
// Both conditions must be true
"rule": "eventType==workitem.updated&&status==Active"

// Either condition can be true
"rule": "action==opened||action==reopened"

// Mixed: (A AND B) OR (C AND D)
"rule": "eventType==created&&status==New||eventType==updated&&status==Active"
```

### Quoted Values

If the expected value contains `&&` or `||` (or you want a single-quoted literal), wrap it in **single quotes**:

```jsonc
"rule": "assignee=='some-user <user@example.com>'"
```

Quotes are optional for simple values. Both of these are equivalent:

```jsonc
"rule": "action==opened"
"rule": "action=='opened'"
```

### JSON Paths

JSON paths use dot notation to traverse the payload. Given:

```json
{ "action": "opened", "pull_request": { "draft": false } }
```

| Expression                  | Result  |
|-----------------------------|---------|
| `action==opened`            | `true`  |
| `action!=closed`            | `true`  |
| `pull_request.draft==false` | `true`  |
| `action==closed`            | `false` |

Type coercion is handled automatically — strings, numbers, booleans, and `null` are compared against the literal on the right-hand side.

#### Property names that contain `.`

If an object **key** contains a dot (common on Azure DevOps, e.g. `System.AssignedTo`), a plain dot-separated path would be ambiguous. Wrap **that segment** in **double quotes** so it is treated as a single property name:

```
resource.fields."System.AssignedTo".newValue
resource.revision.fields."System.Title"
```

Inside a double-quoted segment, a **backslash** escapes the next character (for example if the key itself needed a quote).

This applies to **match** rules and to **`use-inputs`** paths (see below).

#### Arrays: numeric indices

When the value at a path segment is a JSON **array**, a **numeric** segment selects the element at that index (zero-based):

```
items.0.id
resource.reviewers.1.displayName
```

If the index is out of range, the path does not resolve (`==` fails; `!=` treats a missing path as not equal).

#### Arrays: wildcard `*` (match rules only)

For **filter rules** (`match-any`), a path segment `*` means "any element of the array at this point." The prefix before `*` must resolve to an array. The suffix is evaluated against each element until one matches (for positive operators) or none match (for negative operators).

```
resource.reviewers.*.displayName=='xianix-agent'
```

This passes if **any** reviewer object has `displayName` equal to `xianix-agent`. The wildcard works with all six operators:

```
// passes if any label name starts with "hotfix/"
labels.*.name^='hotfix/'

// passes if any message in the thread contains a keyword
comments.*.body*='needs review'
```

Only **one** `*` segment per path is supported.

Wildcard `*` is **not** supported in **`use-inputs`** paths — use a fixed numeric index there if you need a specific array element.

### Operator Examples

**Starts with (`^=` / `!^=`)**

Match a branch that follows a naming convention, or filter events from a specific bot:

```jsonc
// Trigger only for feature branches
"rule": "pull_request.head.ref^=feature/"

// Skip anything pushed by a bot account
"rule": "sender.login!^=bot-"

// Azure DevOps: source branch is a release branch
"rule": "resource.sourceRefName^=refs/heads/release/"
```

**Contains (`*=` / `!*=`)**

Match free-form text fields like commit messages, PR titles, or notification messages:

```jsonc
// Trigger when a PR title signals a breaking change
"rule": "pull_request.title*=BREAKING"

// Azure DevOps: react to specific activity messages
"rule": "message.text*='updated the source branch'"
"rule": "message.text*='as a reviewer'"

// Skip draft descriptions that mention WIP
"rule": "pull_request.body!*=[WIP]"
```

---

## 3. `use-inputs` — Payload Extraction

Extracts values from the webhook payload into named variables. They are used for `execute-prompt` interpolation and are forwarded to the executor (for example as `XIANIX_INPUTS`).

```json
"use-inputs": [
  { "name": "pr-number",      "value": "number" },
  { "name": "repository-url", "value": "repository.clone_url" },
  { "name": "platform",       "value": "github", "constant": true }
]
```

| Field      | Description |
|------------|-------------|
| `name`     | Key in the extracted dictionary |
| `value`    | Dot-separated JSON path into the payload, **or** a literal when `constant` is `true` |
| `constant` | *(optional, default `false`)* When `true`, `value` is used as-is instead of resolving a path |

### Path Resolution Examples

Given:

```json
{
  "number": 42,
  "repository": { "clone_url": "https://github.com/acme/app.git", "full_name": "acme/app" },
  "pull_request": { "title": "Fix auth bug", "head": { "ref": "fix/auth" } }
}
```

| Input definition | Resolved value |
|------------------|----------------|
| `"value": "number"` | `42` |
| `"value": "repository.clone_url"` | `"https://github.com/acme/app.git"` |
| `"value": "pull_request.head.ref"` | `"fix/auth"` |
| `"value": "github", "constant": true` | `"github"` (literal) |
| `"value": "resource.revision.fields.\"System.Title\""` (path uses a quoted segment for a dotted key) | Azure DevOps work item `System.Title` |

If a path does not resolve (missing property), the input is set to `null`.

---

## 4. `use-plugins` — Plugin Installation

Declares Claude Code marketplace plugins to install in the executor container before the prompt runs.

```json
"use-plugins": [
  {
    "plugin-name": "pr-reviewer@xianix-plugins-official",
    "marketplace": "xianix-team/plugins-official",
    "envs": [
      { "name": "GITHUB_PERSONAL_ACCESS_TOKEN", "value": "env.GITHUB_TOKEN" }
    ]
  }
]
```

| Field           | Required | Description |
|-----------------|----------|-------------|
| `plugin-name`   | Yes | Plugin reference in `plugin-name@marketplace-name` form, passed to `claude plugin install` |
| `marketplace`   | No  | Marketplace source (`owner/repo`, git URL, path, or `marketplace.json` URL). Omit for the built-in Anthropic marketplace. |
| `envs`          | No  | Environment variable mappings for this plugin — see below |

### Plugin Environment Variables (`envs`)

The executor container already has a set of variables injected automatically before any plugin runs. The `envs` array lets you **remap or supplement** those variables — the most common reason is that a Claude Code plugin expects a different variable name than the one already present in the container.

#### Variables automatically present in the container

These are always injected by the agent runtime and do not need to be declared in `envs`:

| Variable              | Description |
|-----------------------|-------------|
| `ANTHROPIC_API_KEY`   | Anthropic API key (read directly by the Claude Code SDK) |
| `GITHUB_TOKEN`        | GitHub personal access token — set via `GITHUB_TOKEN` (or `<TENANT>_GITHUB_TOKEN`) in the agent's `.env` |
| `AZURE_DEVOPS_TOKEN`  | Azure DevOps personal access token — set via `AZURE_DEVOPS_TOKEN` (or `<TENANT>_AZURE_DEVOPS_TOKEN`) in `.env` |

#### Why `envs` is needed: renaming for plugins

Claude Code plugins often expect a specific variable name that differs from the one already in the container. Use `envs` to expose the existing value under the name the plugin requires:

```json
{ "name": "GITHUB_PERSONAL_ACCESS_TOKEN", "value": "env.GITHUB_TOKEN" }
```

This reads `GITHUB_TOKEN` from the host environment and makes it available as `GITHUB_PERSONAL_ACCESS_TOKEN` inside the container — so the plugin can find it without any changes to the agent's credential configuration.

#### `env.` reference syntax

By default, `value` is treated as an `env.VAR_NAME` reference. The `env.` prefix is stripped and the named variable is read from the **host** (agent process) environment. Any variable defined in the agent's `.env` file can be referenced this way:

```json
{ "name": "MY_PLUGIN_TOKEN",    "value": "env.GITHUB_TOKEN" }
{ "name": "AZURE_PAT",          "value": "env.AZURE_DEVOPS_TOKEN" }
{ "name": "CUSTOM_SERVICE_KEY", "value": "env.MY_CUSTOM_API_KEY" }
```

If the referenced variable is not set on the host, the injected value will be an empty string.

#### Constant values

Set `"constant": true` to inject a fixed literal string rather than resolving a host variable. This is useful for plugin configuration flags, region identifiers, or any value that does not come from the environment:

```json
{ "name": "REVIEW_MODE",    "value": "strict",    "constant": true }
{ "name": "TARGET_BRANCH",  "value": "main",      "constant": true }
{ "name": "AZURE_ORG_URL",  "value": "https://dev.azure.com/my-org", "constant": true }
```

#### Field reference

| Field      | Description |
|------------|-------------|
| `name`     | Name of the environment variable as it will appear inside the container |
| `value`    | `env.VAR_NAME` to read from the host environment, or a literal string when `constant` is `true` |
| `constant` | *(optional, default `false`)* When `true`, `value` is used as-is without any host variable lookup |

---

## 5. `execute-prompt` — Claude Code Prompt Template

A string template run as the Claude Code prompt after plugins are installed. Use `{{input-name}}` placeholders for resolved `use-inputs` values.

```json
"execute-prompt": "You are reviewing PR #{{pr-number}} titled \"{{pr-title}}\" in {{repository-name}} (branch: {{pr-head-branch}}).\n\nRun /code-review to perform the automated review."
```

Placeholders are replaced case-insensitively. Any `{{name}}` with no matching input is left unchanged.

---

## Complete Example

A rule set with one execution that reviews newly opened GitHub pull requests:

```json
[
  {
    "webhook": "Default",
    "executions": [
      {
        "name": "github-pull-request-review",
        "match-any": [
          { "name": "pr-opened-event", "rule": "action==opened" }
        ],
        "use-inputs": [
          { "name": "pr-number",       "value": "number" },
          { "name": "repository-url",  "value": "repository.clone_url" },
          { "name": "repository-name", "value": "repository.full_name" },
          { "name": "pr-title",        "value": "pull_request.title" },
          { "name": "pr-head-branch",  "value": "pull_request.head.ref" },
          { "name": "platform",        "value": "github", "constant": true }
        ],
        "use-plugins": [
          {
            "plugin-name": "pr-reviewer@xianix-plugins-official",
            "marketplace": "xianix-team/plugins-official",
            "envs": [
              { "name": "GITHUB_PERSONAL_ACCESS_TOKEN", "value": "env.GITHUB_TOKEN" }
            ]
          }
        ],
        "execute-prompt": "You are reviewing pull request #{{pr-number}} titled \"{{pr-title}}\" in the repository {{repository-name}} (branch: {{pr-head-branch}}).\n\nRun /code-review to perform the automated review. The `gh` CLI is authenticated and available if you need it directly."
      }
    ]
  }
]
```

### Azure DevOps example: work item field with a dotted name

Filter when a field whose key contains dots changes (quoted segment):

```jsonc
"rule": "eventType==workitem.updated&&resource.fields.\"System.AssignedTo\".newValue=='xianix-agent <xianix-agent@99x.io>'"
```

### Azure DevOps example: PR updated with a specific reviewer

Require both the event type and the agent in the reviewers list:

```jsonc
"rule": "eventType==git.pullrequest.updated&&resource.reviewers.*.displayName=='xianix-agent'"
```

### Azure DevOps example: PR activity via `contains`

Trigger on specific activity messages such as a source branch push or a reviewer being assigned:

```jsonc
// Source branch was updated
"rule": "eventType==git.pullrequest.updated&&resource.reviewers.*.displayName=='xianix-agent'&&message.text*='updated the source branch'"

// Agent was added as a reviewer
"rule": "eventType==git.pullrequest.updated&&resource.reviewers.*.displayName=='xianix-agent'&&message.text*='as a reviewer'"
```

### Example: branch-convention filter with `starts-with`

Only run the workflow for pull requests targeting a `release/` branch:

```jsonc
"rule": "action==opened&&pull_request.base.ref^=release/"
```

### What Happens at Runtime

1. A webhook fires with a name that matches `webhook` (e.g. `"Default"`).
2. For each execution block, if `match-any` is non-empty, at least one `rule` must pass.
3. `use-inputs` are resolved from the payload.
4. `execute-prompt` is interpolated with those inputs.
5. The executor installs `use-plugins`, applies `envs`, and runs the final prompt.
