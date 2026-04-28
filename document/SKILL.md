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
`/document lang:pt` — force Portuguese output language
`/document lang:en` — force English output language

When `$ARGUMENTS` contains a specific file path or component name, scope the documentation to that component rather than the whole project.

If no argument is given, generate the full suite.

---

## Language

Detect the output language from context in this priority order:
1. Explicit argument: `/document lang:pt`, `/document lang:en`, etc.
2. Language of existing documentation in the project (`docs/`, `README.md`, code comments)
3. Language used by the user in this conversation
4. Default: **English**

All documentation output — diagram labels, section titles, descriptions, impact analysis, fix suggestions, everything — must be in the detected language. Be consistent throughout.

When **Portuguese** is selected, use Portuguese markers: `PERIGOSO`, `NAO_UTILIZADO`, `[DESCONTINUADO]`, `[NÃO UTILIZADO]`.
When **English** is selected, use English markers: `DANGEROUS`, `UNUSED`, `[DISCONTINUED]`, `[UNUSED]`.

---

## Section Titles

Use consistent titles across all files. Adapt to output language:

| File | English | Portuguese |
|------|---------|------------|
| `README.md` | `# <repo-name> — Documentation` | `# <repo-name> — Documentação` |
| `c4-level-1.md` | `# C4 Level 1 — System Context` | `# C4 Nível 1 — Contexto do Sistema` |
| `c4-level-2.md` | `# C4 Level 2 — Container Diagram` | `# C4 Nível 2 — Diagrama de Contêineres` |
| `c4-level-3.md` | `# C4 Level 3 — Component Diagram` | `# C4 Nível 3 — Diagrama de Componentes` |
| `c4-level-4.md` | `# C4 Level 4 — Code Diagrams` | `# C4 Nível 4 — Diagramas de Código` |
| `request-map.md` | `# HTTP Request Map` | `# Mapa de Requisições` |
| `security-issues.md` | `# Security Issues` | `# Problemas de Segurança` |
| `bugs.md` | `# Bugs and Code Quality` | `# Bugs e Problemas de Qualidade` |
| `dead-code.md` | `# Dead Code` | `# Código Morto` |
| `environment-variables.md` | `# Environment Variables` | `# Variáveis de Ambiente` |

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
- **Entry point** — if the system sits behind an API gateway, load balancer, or another system that authenticates and redirects, show it as a separate `subgraph` labeled "Entry Point" (or "Ponto de Entrada" in Portuguese). It is not an actor — it is a system. If users access the system directly, omit this.
- **The system itself** (⚙️) — one box
- **External systems** — third-party APIs, cloud services, data stores, auth providers
- **Arrows** labeled with interaction type

**Actors table:** Columns: `Actor` | `Type` | `Description`
- Description must be **evidence-based** — derived from what the auth code actually validates, not assumed
- Do NOT assign specific roles unless the code explicitly differentiates them
- Examples:
  - ❌ `"Administrator / Editor authenticated via OAuth"` — assigns roles not validated by code
  - ❌ `"Company employee"` — assumes organizational role not checked in code
  - ✅ `"Authenticated via Google OAuth (email domain whitelist)"` — describes what code validates
  - ✅ `"User with valid JWT signed by the auth service"` — evidence-based

**Entry Point table** (if applicable): `System` | `Type` | `Description`

**External Systems table:** `System` | `Description`

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

**⚠️ Discontinued arrow rule — apply at generation time, not as a post-check:** If a container is marked `[DISCONTINUED]`, ALL arrows to/from it must use `-.->` (dashed). Never use `-->` (solid) for a discontinued container. Check dead-code.md before drawing arrows.

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
   - Mark dangerous methods with `DANGEROUS` (⚠️ text equivalent for classDiagram — emoji not supported)
   - Mark unused methods with `UNUSED` (❌ text equivalent for classDiagram — emoji not supported)
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

The `#` column is sequential across all sections.

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
- When the problem is the ABSENCE of something (no auth, no rate limiting, no validation): show the code where the mitigation SHOULD exist but doesn't — e.g., the middleware setup that lacks the auth check, the route handler with no guard, the config file missing the protection
- When the problem is absence of an entire file/mechanism: show the closest entry point (route registration, server setup) to illustrate where the gap is
- End with a summary table: severity | count | main problems

---

### `bugs.md` — Bugs and Code Quality

**Categories:**
- 🔴 **Bugs** — Incorrect behavior, broken features, misleading logic, race conditions, wrong error handling
- 🟠 **Bugs in vestigial code** (optional) — Bugs in dead/unused code that don't affect users today but would if reactivated. Apply this tier when the project has significant dead/unused code that contains its own bugs.
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

Sections are numbered sequentially: `## 1.`, `## 2.`, etc.

Classify each variable by asking in this order:
1. **Used per container** — variable is read by code AND configured somewhere (`.env`, CI/CD, Dockerfile)
2. **Read in code but not configured** — read by code but not found in any config source
3. **Configured but not read** — configured in CI/CD or Dockerfile but the reading code doesn't exist or is commented out
4. **Build-time variables** — CI/CD substitutions per container
5. **Dockerfile variables** — `ENV`/`ARG` set during Docker build

A variable may appear in multiple sections (e.g., a build substitution that maps to a runtime variable). Omit any type with zero variables. Do not misclassify a variable into the wrong type — match the definition exactly.

**Part 2 — Hardcoded values (by category):**

Sections **continue the numbering from Part 1** (e.g., if Part 1 ends at `## 5.`, Part 2 starts at `## 6.`).

Within Part 2 tables, the `#` column uses a **single sequential counter across ALL categories** (1, 2, 3... through the entire Part 2).

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
- **Values that stop being exposed when moved to env var** — server-side values (API keys, passwords, credentials). Currently visible to anyone with git repo access; once moved to CI/CD secrets or a secrets manager, they disappear from source. Note: server-side code is not accessible in the browser, but IS exposed to anyone with repo access — moving to env vars is still important.
- **Values that remain exposed even as env vars** (mark with 🌐) — client-side values that must be rendered in the frontend (browser SDK API keys, `VITE_*` vars, OAuth client IDs used client-side). Moving them to env vars improves configurability but they will **always be visible in the browser**. Add a note explaining why they remain public.

---

## Sensitive Value Handling

**Anonymize sensitive values** in documentation — never include full keys, passwords, tokens, or private keys. Preserve start and end for identification, omit the middle:

- API keys: `AIza...bP20`
- Private keys: `MIIEvg...END PRIVATE KEY`
- Passwords: `RCZ%...])2`
- Service account emails: `name@...iam.gserviceaccount.com`
- Client IDs: `start...end.apps.googleusercontent.com`

Infrastructure identifiers (project IDs, resource IDs, service URLs) are not secrets — they can be kept as-is.

---

## Diagram Style Guide

All diagrams use **Mermaid** syntax. Follow these rules precisely to avoid syntax errors.

---

### Global Rules (all diagram types)

- **Never use single quotes** — always double quotes
- **IDs** (node IDs, participant IDs, entity names): alphanumeric + underscore only (`MY_NODE`, `myNode`, `USER_SESSION`). No spaces, hyphens, dots, slashes, or special characters.
- **Reserved words must NOT be used as IDs**: `end`, `start`, `graph`, `subgraph`, `classDef`, `class`, `note`, `loop`, `alt`, `else`, `par`, `opt`, `rect`, `direction`. Rename them (e.g., `END_STATE`, `START_NODE`).
- **String values with special characters** (parentheses, colons, pipes, brackets, slashes, hyphens) MUST be wrapped in double quotes.
- **No backticks** anywhere in diagram code — they cause parse errors in most Mermaid renderers.

---

### `graph TD` — Context, Container, Component diagrams

**Node format:** Always `NODE_ID["emoji text"]` — square brackets with double-quoted content.
- ✅ `USER["👤 User<br/>Authenticated via OAuth"]`
- ❌ `USER("👤 User")` — round brackets not allowed
- ❌ `USER[👤 User]` — unquoted content causes errors if it contains spaces or special chars
- ❌ `USER['👤 User']` — single quotes not supported

**Node IDs:**
- Alphanumeric + underscore: `MY_BACKEND`, `authService`, `CLOUD_STORAGE`
- No hyphens: use `MY_SERVICE` not `my-service`
- No spaces: `MY_SERVICE` not `my service`
- Each ID must be unique within the diagram — reusing an ID creates an implicit merge

**Node labels:**
```mermaid
graph TD
    ACTOR["👤 Actor Name<br/>Short description"]
    SYSTEM["⚙️ System Name<br/>Technology / Deploy target"]
```
- Use `<br/>` for line breaks inside labels — never `\n` or bare newlines
- Avoid `()` inside quoted labels — parentheses can confuse some renderers

**Arrow labels:** ALWAYS double-quoted — no exceptions:
- ✅ `A -->|"REST + Bearer token"| B`
- ❌ `A -->|REST| B` — bare text breaks on special characters

**Subgraphs:** Quote the name, use a safe ID:
```mermaid
subgraph SG_BACKEND["Backend Services"]
    ...
end
```
- Never use a reserved word as the subgraph ID
- The quoted label after the ID is optional but recommended for readability

**Dead/unused nodes:**
- Add `[UNUSED]` or `[DISCONTINUED]` in the node label text
- Leave disconnected (no arrows) if completely unused
- Use dashed arrows (`-.->`) if the code path exists but is unreachable

**Never use `classDef`** — use emojis for visual differentiation instead.

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

**Arrows:**
- `-->` solid: active connection → `A -->|"REST + Bearer token"| B`
- `-.->` dashed: dead/discontinued connection → `A -.->|"[DISCONTINUED]"| B`
- Always label arrows with interaction type (protocol, data, purpose)

---

### `classDiagram` (C4 Level 4)

⚠️ **Mermaid limitation:** `classDiagram` does NOT support emoji — in class names, method names, property names, or relationship labels. They cause parse errors. This does NOT apply to `graph TD`, `sequenceDiagram`, or `erDiagram`.

**Use plain-text markers only:**
- `DANGEROUS` instead of ⚠️
- `UNUSED` instead of ❌
- Add a legend below every classDiagram: `**Legend:** \`DANGEROUS\` = ⚠️ · \`UNUSED\` = ❌`

**Class names:** PascalCase, no spaces, no special chars, no reserved words (`end`, `class`, etc.)

**Method format:**
```mermaid
classDiagram
    class AuthService {
        +login(credentials: Credentials): Token
        -validateToken(token: string): boolean DANGEROUS
        +logout(): void
        +refreshToken(): Token UNUSED
        +userId: string
    }
```
- Visibility: `+` public, `-` private, `#` protected, `~` package
- Return type goes after `)`: `+method(param: Type): ReturnType`
- Markers go after return type: `+method(): Type DANGEROUS`

**Relationship targets MUST be defined class names — never quoted strings:**
- ✅ `class SomeUtil { }` then `ClassA --> SomeUtil : uses`
- ❌ `ClassA --> "SomeUtil" : uses` — quoted strings as targets cause parse errors

**Stereotypes:** `<<interface>>`, `<<abstract>>`, `<<service>>`, `<<enum>>` — double angle brackets are supported.

**Avoid very long class blocks** — Mermaid has rendering limits. Split large classes across multiple diagrams if needed.

**Relationships:**
- `-->` uses / depends on
- `--|>` inherits / extends
- `*--` composition
- `o--` aggregation
- `..>` realizes / implements (interface)

---

### `sequenceDiagram` (C4 Level 4)

**Participant IDs:** Short alphanumeric aliases only. Put the full display name in the `as` clause:
```mermaid
sequenceDiagram
    participant U as 👤 User
    participant FE as 🌐 Frontend
    participant BE as ⚙️ Backend
    participant DB as 📊 Database
```
- Emoji are safe in `sequenceDiagram` participant `as` labels
- IDs must be short: `U`, `FE`, `BE`, `DB` — not `Frontend_Service` or `Auth Backend`

**Message text:** After `:`, plain text is safe. Avoid backticks and nested double quotes.

**`Note over`:** Use participant aliases (short IDs), comma-separated:
- ✅ `Note over FE,BE: Important note`
- ❌ `Note over 🌐 Frontend, ⚙️ Backend: note` — use aliases, not display names

**`alt`/`else`/`end`:** Condition text after `alt` is plain text — no quotes, no backticks:
```mermaid
alt Token valid
    BE-->>FE: 200 OK
else Token expired
    BE-->>FE: 401 Unauthorized
end
```

**Arrows:**
- `->>` for requests/calls
- `-->>` for responses
- `->` for calls without arrowhead (rare)

**Loops:** `loop Description` then steps then `end`

**Activation:** Use `+`/`-` prefix or explicit `activate`/`deactivate` — both work:
```mermaid
U->>+FE: Click submit
FE->>+BE: POST /api/data
BE-->>-FE: 200 OK
FE-->>-U: Show result
```

---

### `erDiagram` (C4 Level 4)

**Entity names:** UPPER_CASE, no spaces, underscores OK:
- ✅ `USER_SESSION`, `PRODUCT`, `ORDER_ITEM`
- ❌ `User Session`, `user-session`

**Attribute format:**
```mermaid
erDiagram
    USER {
        uuid id PK "primary key"
        string email "unique"
        string password_hash
        datetime created_at
        boolean is_active
    }
```
- Format: `TYPE column_name` or `TYPE column_name PK "description"` or `TYPE column_name FK "description"`
- The quoted description is optional
- Use standard types: `string`, `int`, `bigint`, `boolean`, `datetime`, `float`, `text`, `uuid`, `json`
- Avoid exotic types Mermaid may not recognize

**Relationship labels:** ALWAYS in double quotes:
- ✅ `TABLE_A ||--o{ TABLE_B : "has many"`
- ❌ `TABLE_A ||--o{ TABLE_B : has many` — unquoted labels with spaces cause errors

**Relationship cardinality syntax:**
- `||--||` one-to-one (exactly one on both sides)
- `||--o{` one-to-many (zero or more)
- `||--|{` one-to-many (one or more)
- `}o--o{` many-to-many
- `|o--o|` zero or one on both sides

**Include all data stores** — not just relational databases. NoSQL collections, analytics tables, key-value stores all get entities in `erDiagram`.

---

## Cross-Document Rules

**Consistency:** The status of an item must be consistent across all docs.
- If listed in `dead-code.md` → must NOT be `✅ Active` in `c4-level-3.md` component tables
- If listed in `dead-code.md` → must have `UNUSED` marker in `c4-level-4.md` classDiagram methods
- If a discontinued container/system → must have `[DISCONTINUED]` label AND all arrows `-.->` in `c4-level-2.md`
- If listed as dead in `request-map.md` → must not appear as active in C4 diagrams
- `dead-code.md` is the source of truth for what is unused — check it before setting any status

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
- API keys (`AIzaSy*`, `sk-*`, `hf_*`, `ya29.*`, etc.)
- Private keys (`-----BEGIN PRIVATE KEY-----`)
- Passwords and bearer tokens
- Service account / IAM emails — full email must be truncated to `name@...domain.com`
- Client IDs — full OAuth client IDs must be truncated

### 2. Cross-consistency check
For every item in `dead-code.md`:
- In `c4-level-3.md` component tables → must NOT be `✅ Active`
- In `c4-level-4.md` classDiagram → must have `UNUSED` marker
- If a container/system → in `c4-level-2.md` must have `[DISCONTINUED]` label and all arrows must be `-.->` (dashed)

### 3. Code snippet check
Every `### N.` issue in `security-issues.md` and `bugs.md` must have a code block between `**File:**` and `**Risk:**`/`**Impact:**`. No exceptions.

When the problem is the ABSENCE of something (no auth check, no rate limiting, no input validation): show the code where the mitigation SHOULD exist but doesn't — the middleware setup, the route handler, the config file where the protection is missing. If the entire mechanism is absent, show the closest entry point (route registration, server setup) to illustrate the gap.

### 4. Cross-reference check
- Every security issue with bug overlap → references `bugs.md`
- Every committed credential/env file → references `environment-variables.md`

### 5. Section completeness
- `request-map.md`: all 8 section types were evaluated. If a category has no items, include the header with a "None" note — except clearly inapplicable sections (e.g., "Embedded content" can be omitted if there are no external scripts)
- `environment-variables.md`: all 5 Part 1 types were evaluated. No variable was misclassified. Part 2 section numbers continue from Part 1. The `#` column in Part 2 is a single sequential counter across all categories.

---

## Development-Time Usage Patterns

This skill works during active development, not just as a post-hoc audit. Common patterns:

**Design phase:** Run `/document c4` before writing code — document the intended architecture. Diagrams become a contract for implementation.

**Pre-commit check:** Run `/document security` before committing — catch hardcoded secrets, missing auth, injection risks early.

**Sprint review:** Run `/document update` after a sprint — diff existing docs against the current codebase and update what changed.

**New component:** Run `/document c4` scoped to a specific folder or component name to document just that piece.

**Security audit:** Run `/document security bugs` to generate a combined security and quality report for review.

**Onboarding:** Run `/document` once to generate the full suite — gives new team members a navigable map of the codebase immediately.
