---
name: document
description: Generate comprehensive C4 architecture documentation with security analysis, bug tracking, dead code detection, request mapping, and environment variable inventory for any software project. Works incrementally during development or as a full audit on existing codebases.
---

# Project Documentation Skill

Generate structured, evidence-based technical documentation for any software project — frontend, backend, fullstack, microservices, monolith, cloud-native, or on-premise. Can produce a full documentation suite or targeted reports. Works for new projects being built, existing codebases being audited, or evolving systems being maintained.

## Arguments

`/document` — full documentation suite (all files)
`/document c4` — C4 architecture diagrams only (all 4 levels)
`/document security` — security analysis only
`/document dead-code` — unused code detection only
`/document env` — environment variable inventory only
`/document bugs` — bugs and code quality issues only
`/document requests` — HTTP request map only
`/document update` — update existing docs to reflect recent code changes (diff existing docs against current code, update what changed)

When `$ARGUMENTS` contains a specific file path or component name, scope the documentation to that component rather than the whole project.

If no argument is given, generate the full suite.

---

## Language

Detect the output language from context in this priority order:
1. Explicit argument: `/document lang:pt`, `/document lang:es`, etc.
2. Language of existing documentation in the project (`README.md`, `docs/`, comments)
3. Language used by the user in this conversation
4. Default: **English**

All documentation output — diagram labels, section titles, descriptions, impact analysis, fix suggestions, everything — must be in the detected language. Be consistent throughout.

---

## Before You Start: Codebase Exploration

Read the project before generating anything. Follow this priority order:

### Priority 1 — Architecture Understanding

| What to read | Why | Where to look |
|---|---|---|
| Entry points & routes | Maps the API surface | `routes.*`, `main.*`, `app.*`, `index.*`, `server.*`, framework-specific route files |
| Controllers / handlers | Business logic, data flow | `controllers/`, `handlers/`, `views/`, route handler functions |
| Auth middleware | Security model | `middleware/`, `middlewares/`, auth decorators, guards |
| Data models | Data structure | `models/`, ORM definitions, schema files, migrations |
| Service connectors | External integrations | API clients, SDK initializations, external service adapters |
| Frontend pages / screens | User flows | `pages/`, `screens/`, `views/`, `app/` directories |
| Frontend API layer | What the frontend calls | `services/`, `api/`, `hooks/`, `store/` directories |
| Infrastructure config | Deployment model | `Dockerfile`, `docker-compose.yml`, `k8s/`, `terraform/`, `*.yaml` |
| CI/CD config | Build and deploy pipeline | `.github/workflows/`, `cloudbuild.yaml`, `Jenkinsfile`, `.gitlab-ci.yml` |

### Priority 2 — Security & Quality

| What to read | Why | Where to look |
|---|---|---|
| Environment files | Committed secrets | `.env`, `.env.*`, `.env.example`, `.envrc` |
| Credential files | Private keys in git | `credentials.json`, `*.pem`, `*.key`, any JSON with `private_key` |
| Config / settings | Hardcoded values | `settings.*`, `config/`, `constants.*`, `config.js`, `appsettings.json` |
| CORS configuration | Access control | Any `cors`, `Access-Control-*`, `allowedOrigins` setup |
| Query construction | Injection risks | SQL/NoSQL string interpolation, raw query builders |
| Token / session handling | Auth vulnerabilities | JWT handling, session storage, token in URLs, cookie config |

---

## Output Structure

Place `README.md` at the **project root**. All other docs go inside a `docs/` folder:

```
README.md                      # Index linking to all docs (project root)
docs/
├── c4/
│   ├── c4-level-1.md          # System Context
│   ├── c4-level-2.md          # Container Diagram
│   ├── c4-level-3.md          # Component Diagram
│   └── c4-level-4.md          # Code Diagram (classes + sequences + data)
├── request-map.md             # Every HTTP request the system makes
├── security-issues.md         # Vulnerabilities by severity
├── bugs.md                    # Bugs and code quality issues
├── dead-code.md               # Unused files, functions, dependencies
└── environment-variables.md   # Env vars by type + hardcoded values
```

---

## Generation Order

Always generate in this order to satisfy cross-document dependencies:

1. **`dead-code.md`** — source of truth for what is unused
2. **`security-issues.md`** — source of truth for vulnerabilities
3. **`bugs.md`** — may reference `security-issues.md`
4. **`environment-variables.md`** — references `security-issues.md` and `dead-code.md`
5. **`request-map.md`** — references `dead-code.md` for dead requests
6. **`c4-level-1.md`** → **`c4-level-4.md`** — references `dead-code.md` for status markers
7. **`README.md`** — index at project root, generated last

---

## How to Write Each File

### `README.md` — Index

**Title:** `# <repo-name> — Documentation` (or equivalent in the output language)

**Content:**
- 1–2 sentence description of what the project does
- Index table: Document (link) | Description (one line)
- Keep it concise — no architecture details here (those go in C4)

---

### `c4/c4-level-1.md` — System Context

**What it shows:** The system as a black box — who uses it, what external systems it talks to.

**Diagram:** Mermaid `graph TD` with:
- **Actors** (👤) — who uses the system
- **Entry point** — if the system sits behind an API gateway, load balancer, or another system that authenticates and redirects, show it as a separate `subgraph` labeled "Entry Point" (it is not an actor, it is a system). If users access the system directly, omit this.
- **The system itself** (⚙️) — one box
- **External systems** — third-party APIs, cloud services, data stores, auth providers
- **Arrows** labeled with interaction type

**Actors table:** Columns: `Actor`, `Type`, `Description`
- Description must be **evidence-based** — derived from what the auth code actually validates, not assumed
- Do NOT assign specific roles unless the code explicitly differentiates them
- Examples:
  - ❌ `"Administrator / Editor authenticated via OAuth"` — assigns roles not validated by code
  - ✅ `"Authenticated via Google OAuth (email domain whitelist)"` — describes what code validates
  - ✅ `"User with valid JWT signed by auth service"` — evidence-based

**Entry Point table** (if applicable): `System`, `Type`, `Description`

**External Systems table:** `System`, `Description`

**Main Flow:** Numbered list (5–10 steps) describing the main happy path end-to-end. High-level — no file names or implementation details. Must be consistent with the diagram.

---

### `c4/c4-level-2.md` — Container Diagram

**What it shows:** Deployable units (frontend, backend APIs, databases, functions, queues) and how they communicate.

**Diagram:** Mermaid `graph TD` with `subgraph` blocks for:
- Frontend(s)
- Backend API(s)
- Background workers / serverless functions
- Data stores
- External services

Arrows show protocols (REST, gRPC, WebSocket, Pub/Sub, AMQP, etc.)

**Important:** If a container is marked `[DISCONTINUED]`, ALL arrows to/from it must use `-.->` (dashed). Apply at generation time.

**Container table:** Name | Technology | Deployment target | Description

---

### `c4/c4-level-3.md` — Component Diagram

**What it shows:** Internal components of each major container.

**Diagram:** One Mermaid `graph TD` per major container showing:
- Controllers / routes
- Services / use cases
- Repositories / data access
- Middleware
- Internal dependencies (arrows between components)
- For frontend: pages, components, hooks, services, state management

**Component table** below each diagram: `Component` | `File path` | `Description` | `Status`

**Status decision tree:**
1. Entire component listed in `dead-code.md` as unused → `❌ Unused`
2. Component has methods listed in `dead-code.md` as unused, or is partially implemented → `⚠️ Partially active` with note and cross-reference to `dead-code.md`
3. Otherwise → `✅ Active`

---

### `c4/c4-level-4.md` — Code Diagram

**What it shows:** Class structures, key interaction sequences, and data schemas.

**Content:**
1. **`classDiagram`** — main classes/modules with methods and properties
   - Mark dangerous methods with `DANGEROUS` (⚠️ text equivalent for classDiagram)
   - Mark unused methods with `UNUSED` (❌ text equivalent for classDiagram)
   - Show relationships: `-->` (uses), `--|>` (inherits), `*--` (composition)
   - Add a legend: `**Legend:** \`DANGEROUS\` = ⚠️ · \`UNUSED\` = ❌`
   - ⚠️ classDiagram does NOT support emoji in method/class names — use plain text markers only

2. **`sequenceDiagram`** — for key flows:
   - Main happy path
   - Authentication / authorization flow
   - Any async or event-driven flow

3. **`erDiagram`** — for ALL data stores (relational, document, analytics, key-value)

---

### `request-map.md` — HTTP Request Map

**What it shows:** Every HTTP request the system makes, organized by origin.

**Sections:**
1. Frontend → Backend API
2. Backend → External Services
3. Backend → Database queries (when query shape matters)
4. Serverless / Background → External APIs
5. Frontend → External Services (direct browser calls)
6. Frontend redirects (navigation, not API calls)
7. Dead / unused requests
8. Embedded content — iframes, external scripts loaded (if applicable)

**Columns:** `#` | `Method` | `URL` | `Source file` | `Purpose` | `Auth`

**Summary:** Status summary table at the end counting active, dead, placeholder, and vestigial requests.

---

### `security-issues.md` — Security Issues

**Severity levels:**
- 🔴 **CRITICAL** — Immediate exploitation risk (credentials committed, SQL injection, unauthenticated endpoints exposing sensitive data)
- 🟠 **HIGH** — Significant risk requiring action (client-only auth, exposed internals, missing auth on processing endpoints)
- 🟡 **MEDIUM** — Notable concerns (no CSRF, wildcard CORS, insecure storage, no rate limiting, XSS risks)
- 🔵 **LOW** — Minor issues (debug logging in production, error detail leakage, dependency version exposure)

**Each issue format:**

```markdown
### N. Descriptive title

**File:** `path/to/file.ext:42`

```lang
// code snippet showing the problem
```

**Risk:** Description of the risk.

**Fix:** Suggested remediation.

---
```

**Rules:**
- Issues are numbered **globally** (1, 2, 3... across all severity tiers — not reset per tier)
- **Every issue must have a code snippet — no exceptions**
- When the problem is the ABSENCE of something (no auth, no rate limiting), show the code where the mitigation SHOULD exist but doesn't
- End with a summary table: severity | count | main problems

---

### `bugs.md` — Bugs and Code Quality

**Categories:**
- 🔴 **Bugs** — Incorrect behavior, broken features, misleading logic, race conditions, wrong error handling
- 🟠 **Bugs in vestigial code** (optional) — Bugs in dead/unused code that don't affect users today but would if reactivated
- 🟡 **Code Quality Issues** — Works but problematic: duplicate logic, inconsistencies, API convention violations, misleading names

**Each issue format:**

```markdown
### N. Descriptive title

**File:** `path/to/file.ext:42`

```lang
// code snippet showing the problem
```

**Impact:** Description of the impact.

---
```

---

### `dead-code.md` — Dead Code

**This is the source of truth for all other docs.** Anything listed here must be marked as unused/discontinued everywhere else.

**Sections to look for:**
1. Unused files (not imported anywhere)
2. Dead imports (importing missing or never-used modules)
3. Unreachable functions (defined but never called)
4. Commented-out code blocks (with line ranges)
5. Partially implemented features (code exists but is incomplete or disabled)
6. Unused dependencies (`package.json`, `requirements.txt`, `go.mod`, etc.)
7. Misplaced artifacts (copy-pasted from other projects, IDE config committed to repo)
8. Discontinued systems (entire features removed but code remains)

---

### `environment-variables.md` — Environment Variables

**Organization:** By **type of variable**, not by component. Two main blocks separated by `---`.

**Part 1 — Environment variables (by type):**

Classify each variable by asking in this order:
1. **Used per container** — variable is read by code AND configured somewhere (`.env`, CI/CD, Dockerfile)
2. **Read in code but not configured** — read by code but not found in any config source
3. **Configured but not read** — configured in CI/CD or Dockerfile but the reading code doesn't exist or is commented out
4. **Build-time variables** — CI/CD substitutions per container
5. **Dockerfile variables** — `ENV`/`ARG` set during Docker build

A variable may appear in multiple sections (e.g., a build substitution that maps to a runtime variable). Omit any type with zero variables.

**Part 2 — Hardcoded values (by category):**

Only include categories that apply to the project. Common categories:
1. 🔴 API keys and secrets (always first if present)
2. Service URLs and infrastructure endpoints
3. Cloud / hosting project identifiers
4. Database names and connection details
5. Storage bucket / container names
6. AI / ML model names and parameters
7. Regions and availability zones
8. Processing parameters (thresholds, timeouts, limits)
9. Access control lists (email or domain allowlists)
10. Other project-specific categories as needed

**For each hardcoded value:** Current value | File:line | Suggested env var name

**Exposure distinction:** For each hardcoded value, distinguish:
- **Values that would stop being exposed if moved to env var** — server-side values (API keys, passwords, credentials) that are currently in source but would stop being visible once moved to CI/CD secrets or a secrets manager
- **Values that remain exposed even as env vars** — client-side values rendered in the frontend (API keys for browser SDKs, public config). Mark with 🌐 and explain why they remain public

---

## Sensitive Value Handling

**Anonymize sensitive values** in documentation — never include full keys, passwords, tokens, or private keys. Preserve start and end for identification, omit the middle:

- API keys: `AIza...bP20`
- Private keys: `MIIEvg...END PRIVATE KEY`
- Passwords: `RCZ%...])2`
- Service account emails: `name@...iam.gserviceaccount.com`

Infrastructure identifiers (project IDs, resource IDs, service URLs) are not secrets — they can be kept as-is.

---

## Diagram Style Guide

All diagrams use **Mermaid** syntax.

### `graph TD` — Context, Container, Component diagrams

**Node format:** Always `NODE_ID["emoji text"]` (square brackets with quotes)
- Never use `classDef` + `class` for colors — use emojis instead
- Never use round brackets `("text")` — always square `["text"]`
- Never use bare text without emoji

**Node labels:**
```mermaid
graph TD
    ACTOR["👤 Actor Name<br/>Short description"]
    SYSTEM["⚙️ System Name<br/>Technology / Deploy target"]
```

**Emoji mapping (consistent across all diagrams):**
- 👤 Person / Actor / User
- 🌐 Web frontend / browser client
- 📱 Mobile client
- ⚙️ System / Backend / Worker / Function
- 🔐 Auth service / Identity provider
- 📁 File storage (S3, GCS, Blob)
- 📊 Data store (SQL, NoSQL, Data Warehouse)
- 🧠 AI / ML service
- 💬 Messaging / Chat / Queue
- 📄 Document / Page / Report
- 🧩 Component / Module
- 📡 Route / API endpoint
- 🔀 Gateway / Load balancer / Proxy

**Grouping:**
```mermaid
graph TD
    subgraph "Group Name"
        A["⚙️ Service A"]
        B["⚙️ Service B"]
    end
```

**Arrows:**
- `-->` solid: active connection → `A -->|"REST + Bearer token"| B`
- `-.->` dashed: dead/discontinued connection → `A -.->|"[DISCONTINUED]"| B`
- Always label arrows with interaction type (protocol, data, purpose)

**Dead/unused nodes:**
- Add `[UNUSED]` or `[DISCONTINUED]` in the node label
- Leave disconnected (no arrows) if completely unused
- Use dashed arrows (`-.->`) if the code path exists but is unreachable

### `classDiagram` (C4 Level 4)

⚠️ **Mermaid limitation:** `classDiagram` does NOT support emoji — they cause parse errors. Use plain-text markers instead. This does NOT apply to `graph TD`, `sequenceDiagram`, or `erDiagram`.

```mermaid
classDiagram
    class ClassName {
        +publicMethod(param) ReturnType
        -privateMethod() DANGEROUS
        +unusedMethod() UNUSED
        +property: Type
    }
    ClassA --> ClassB : uses
```

Always define dependency classes as proper `class` blocks — do NOT use quoted strings as targets (`ClassA --> "someUtil"` causes parse errors).

### `sequenceDiagram` (C4 Level 4)

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant FE as 🌐 Frontend
    participant BE as ⚙️ Backend
    participant DB as 📊 Database

    U->>FE: User action
    FE->>BE: POST /endpoint (payload)
    BE->>DB: Query
    DB-->>BE: Result
    BE-->>FE: 200 OK + data
```

- Use `->>` for requests, `-->>` for responses
- Use `alt`/`else` for conditional flows
- Use `loop` for iterations
- Use `Note over A,B:` for important annotations

### `erDiagram` (C4 Level 4)

```mermaid
erDiagram
    TABLE_NAME {
        TYPE column_name PK "description"
        TYPE column_name FK "description"
        TYPE column_name "description"
    }
    TABLE_A ||--o{ TABLE_B : "relationship"
```

---

## Cross-Document Rules

**Consistency:** The status of an item must be consistent across all docs.
- If listed in `dead-code.md` → must NOT be `✅ Active` in `c4-level-3.md`
- If listed as dead in `request-map.md` → must not appear as active in C4 diagrams
- `dead-code.md` is the source of truth for what is unused

**Cross-references:** Add them when relevant:
- Security issues that overlap with bugs → reference `bugs.md`
- Committed `.env` or credential files in `security-issues.md` → reference `environment-variables.md`
- Unused dependencies in `dead-code.md` → note in `environment-variables.md`

**Evidence-based:** Every claim must be backed by code, configuration, or logs. Never assume — if something is ambiguous, ask before documenting.

---

## Post-Generation Validation

After generating all files, run these checks before delivering:

### 1. Anonymization sweep
Scan every generated file for full values of:
- API keys (`AIzaSy*`, `sk-*`, `hf_*`, etc.)
- Private keys (`-----BEGIN PRIVATE KEY-----`)
- Passwords and bearer tokens
- Service account / IAM emails (full email must be truncated)
- Client IDs (full OAuth client IDs must be truncated)

### 2. Cross-consistency check
For every item in `dead-code.md`:
- In `c4-level-3.md` component tables → must NOT be `✅ Active`
- In `c4-level-4.md` classDiagram → must have `UNUSED` marker
- If a container/system → in `c4-level-2.md` must have `[DISCONTINUED]` label and all arrows must be `-.->` (dashed)

### 3. Code snippet check
Every `### N.` issue in `security-issues.md` and `bugs.md` must have a code block. No exceptions.

### 4. Cross-reference check
- Every security issue with bug overlap → references `bugs.md`
- Every committed credential/env file → references `environment-variables.md`

### 5. Section completeness
- `request-map.md`: all 8 section types were evaluated (include header + "None" if a section has no items, except omit empty ones that clearly don't apply)
- `environment-variables.md`: all 5 Part 1 types were evaluated; no variable was misclassified

---

## Development-Time Usage Patterns

This skill is designed to work during active development, not just as a post-hoc audit. Common patterns:

**Design phase:** Run `/document c4` before writing code — document the intended architecture. Diagrams become a contract for implementation.

**Pre-commit check:** Run `/document security` before committing — catch hardcoded secrets, missing auth, injection risks early.

**Sprint review:** Run `/document update` after a sprint — diff existing docs against the current codebase and update what changed.

**New component:** Run `/document c4` scoped to a specific folder or component name to document just that piece.

**Security audit:** Run `/document security bugs` to generate a combined security and quality report for review.

**Onboarding:** Run `/document` once to generate the full suite — gives new team members a navigable map of the codebase immediately.
