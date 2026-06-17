---
name: c4-diagram
description: Creates and refines C4 architecture diagrams using PlantUML C4 syntax. Trigger this skill whenever the user mentions C4 diagrams, architecture diagrams, system context diagrams, container diagrams, or component diagrams — or when they describe a system and want to visualize its architecture. Also trigger when they say things like "create a C4 diagram of this", "draw the architecture", "visualize our system", "update the diagram based on above", "improve the diagrams", or "add X to the diagram". Even if the user doesn't say "C4" explicitly, use this skill any time they want a formal architecture diagram. Always use this skill for architecture visualization tasks.
---

# C4 Diagram Skill

## Step 1: Understand what to diagram

**From conversation:** Diagram exactly what was described — don't add databases, queues, or other infrastructure that wasn't mentioned. If something seems architecturally missing, ask.

**From codebase or docs:** Read the relevant files to discover the real architecture before writing anything:
- `docker-compose.yml`, `kubernetes/`, `helm/` — services and relationships
- `README.md`, `docs/`, `architecture/` — stated intent
- `package.json`, `requirements.txt`, `go.mod` — external dependencies
- Config files, `.env.example`, `openapi.yaml` — integrations and APIs

**Updating an existing diagram:** Start from the current PlantUML code in context and apply the changes.

**Which level(s) to create:**

| Level | Include | Shows |
|-------|---------|-------|
| 1: Context | `C4_Context.puml` | System + external users and systems |
| 2: Container | `C4_Container.puml` | Apps, databases, queues inside the system |
| 3: Component | `C4_Component.puml` | Modules/services inside one container |

Default to Level 1 + Level 2 when not specified.

## Step 2: Write the PlantUML C4 code

```plantuml
@startuml
!include <C4/C4_Context.puml>

title System Context diagram for [System Name]

Person(customer, "Customer", "Browses and purchases products.")
System(store, "E-Commerce Platform", "Core shopping experience.")
System_Ext(stripe, "Stripe", "Payment processing.")
System_Ext(auth0, "Auth0", "Authentication via OIDC.")

Rel(customer, store, "Browses, orders, pays")
Rel(store, stripe, "Processes payments", "HTTPS/REST")
Rel(store, auth0, "Authenticates users", "OIDC")

Lay_R(store, stripe)

LAYOUT_WITH_LEGEND()
@enduml
```

**Argument counts:** `System`, `System_Ext`, `Person`, `Person_Ext` take **3 args**: `(alias, "Label", "Description")`. `Container`, `ContainerDb`, `ContainerQueue` and their `_Ext` variants take **4 args** including a technology field. Never pass `""` for a missing field — it garbles the output.

**Layout directives:**
- `LAYOUT_WITH_LEGEND()` — always include this
- `LAYOUT_LANDSCAPE()` — use when many elements need to spread horizontally
- `Lay_R(a, b)` — `a` left of `b`
- `Lay_D(a, b)` — `a` above `b`

Declare elements in reading order (users first, external systems last) and add `Lay_R`/`Lay_D` hints for any elements that still clump.

**Quality rules:**
- Only include what you can verify — from the conversation or from the code.
- If a service has a database that wasn't mentioned, note it in the service description rather than adding a node. Only draw it separately if it was explicitly called out or found in config files.
- Aim for ~12 elements per diagram, hard cap at 20 — split or go to the next C4 level if you need more.
- Use unidirectional arrows only (`Rel`, not `BiRel`) — bidirectional arrows obscure who initiates the interaction.
- Label relationships with action verbs and a technology: `"Sends order confirmation", "SMTP"` not `"uses"`.
- Use `_Ext` variants for everything outside the system boundary.
- Descriptions: one short sentence for a non-technical reader, under 50 characters where possible.
- For containers: always include the technology field.
- Use descriptive aliases — `orderService` not `s1`.

## Step 3: Render and inspect

Save to a `.puml` file, then render:

```python
python -c "
import base64, zlib, urllib.request, sys
src = open(sys.argv[1], 'r', encoding='utf-8').read()
enc = base64.urlsafe_b64encode(zlib.compress(src.encode('utf-8'), 9)).decode('ascii')
url = f'https://kroki.io/plantuml/png/{enc}'
req = urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})
with urllib.request.urlopen(req) as r, open(sys.argv[2], 'wb') as f:
    f.write(r.read())
" c4_context.puml c4_context.png
```

View the PNG with your Read tool and check for:
- Overlapping labels or arrows
- Elements clumped in one area
- Uncrossable arrow paths
- Truncated text

| Problem | Fix |
|---------|-----|
| Labels overlap | Add `Lay_R(a, b)` to push elements apart |
| Everything vertical | Add `LAYOUT_LANDSCAPE()` |
| Arrows cross | Reorder declarations; add more `Lay_` hints |
| Too crowded | Remove least important elements or mention them in descriptions |

Iterate until clean.

## Step 4: Save and summarize

Save files to `docs/architecture/` if the project has a `docs/` folder, otherwise the current working directory. Use this naming convention:

| Diagram | Files |
|---------|-------|
| System context | `c4-context.puml` + `c4-context.png` |
| Containers | `c4-containers.puml` + `c4-containers.png` |
| Components (per feature) | `c4-components-{feature}.puml` + `.png` |
| Deployment | `c4-deployment.puml` + `c4-deployment.png` |
| Dynamic flow | `c4-dynamic-{flow}.puml` + `.png` |

To embed a diagram in GitHub markdown, add this to the relevant README or doc:
```markdown
![System Context](docs/architecture/c4-context.png)
```

Tell the user where the files are saved and give one sentence per diagram describing what it shows.

## Reference: Level 1 example

```plantuml
@startuml
!include <C4/C4_Context.puml>

title System Context diagram for E-Commerce Platform

Person(customer, "Customer", "Browses and purchases products online.")
Person(admin, "Store Admin", "Manages products, orders, and promotions.")

System(store, "E-Commerce Platform", "Core shopping experience and order management.")

System_Ext(stripe, "Stripe", "Payment processing.")
System_Ext(sendgrid, "SendGrid", "Transactional email.")
System_Ext(auth0, "Auth0", "Authentication via OIDC.")

Rel(customer, store, "Browses, orders, pays")
Rel(admin, store, "Manages catalogue and orders")
Rel(store, stripe, "Processes payments", "HTTPS/REST")
Rel(store, sendgrid, "Sends receipts", "HTTPS/REST")
Rel(store, auth0, "Authenticates users", "OIDC")

Lay_R(store, stripe)
Lay_R(store, sendgrid)
Lay_D(stripe, sendgrid)

LAYOUT_WITH_LEGEND()
@enduml
```

## Reference: Level 2 example

```plantuml
@startuml
!include <C4/C4_Container.puml>

title Container diagram for E-Commerce Platform

Person(customer, "Customer", "Browses and purchases products.")

System_Boundary(platform, "E-Commerce Platform") {
    Container(frontend, "Web Frontend", "React 18", "Serves the shopping experience.")
    Container(api, "API Server", "Node.js / Express", "Business logic and data access.")
    ContainerDb(db, "Database", "PostgreSQL 15", "Products, orders, and accounts.")
}

System_Ext(stripe, "Stripe", "Payment processing.")
System_Ext(sendgrid, "SendGrid", "Transactional email.")
System_Ext(auth0, "Auth0", "Authentication.")

Rel(customer, frontend, "Browses and shops", "HTTPS")
Rel(frontend, api, "Calls", "HTTPS/REST")
Rel(api, db, "Reads and writes", "SQL/TLS")
Rel(api, stripe, "Charges card", "HTTPS/REST")
Rel(api, sendgrid, "Sends email", "HTTPS/REST")
Rel(api, auth0, "Validates token", "OIDC")

Lay_R(api, stripe)
Lay_D(api, db)

LAYOUT_WITH_LEGEND()
@enduml
```
