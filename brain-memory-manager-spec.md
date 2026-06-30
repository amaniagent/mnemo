# Brain Memory Manager — Specification (Work in Progress)

> **Status:** In Ausarbeitung — sequentielle Klärung via `sequential-clarification-spec` Skill
> **Letzte Aktualisierung:** 2025-06-29
> **Basierend auf:** `/home/jay/system/files/brain-memory manager.md` (Original-Draft)

---

## 1. Overview & Guiding Principles

**Zweck:** Mehrschichtiges Memory-System, das Informationen sinnvoll auf verschiedene Speicher-Layer verteilt & synchronisiert, damit jeder Agent mit minimalem Context/Token-Verbrauch das bestmögliche Ergebnis erreicht.

**Prinzipien:**
- **Layer-Isolation** — Jeder Layer hat klaren Zweck, Ownership & Zugriffsregeln
- **Token-Minimierung** — Agents laden nur ihr relevantes Subset (via Tags, Namespaces, MCP)
- **Human-Readable** — Markdown als Primaerformat, JSON für Indizes
- **Plug&Play** — Gesamtes System als Package installierbar (spätere Vision)
- **Observability** — Langfuse-Integration für Tracing, Evaluation, Privacy-Control

---

## 2. Layer Architecture (6 Layer)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER-MEMORY          → User-Profile, Preferences, History   │
├─────────────────────────────────────────────────────────────────┤
│ 2. AGENT-MEMORY         → Skills, MCPs, CLIs, Agent-Configs    │
├─────────────────────────────────────────────────────────────────┤
│ 3. SESSION-MEMORY       → Raw Sessions + PageIndex Analytics   │
├─────────────────────────────────────────────────────────────────┤
│ 4. PROJECT-MEMORY       → Project Goals, Stack, ADRs, TODOs    │
├─────────────────────────────────────────────────────────────────┤
│ 5. KNOWLEDGE-MEMORY     → Repos, Wiki, RAG (persistentes Wissen)│
├─────────────────────────────────────────────────────────────────┤
│ 6. LONGTERM-MEMORY      → Promovierte Session-Insights (NEU)   │
└─────────────────────────────────────────────────────────────────┘
```

### Layer-Flüsse (entschieden)

| Von | Nach | Mechanismus |
|-----|------|-------------|
| `session-memory` | `longterm-memory` | **Hybrid Promotion** — Auto-Score als Vorschlag + Human-Confirm (One-Click) — Default: "Später entscheiden" — **Nach 28 Tagen Inaktivität: Auto-Promotion (kein Delete)** |
| `project-memory` | `knowledge-memory/wiki` | Referenzierung (Repos gespiegelt in `knowledge-memory/repo/`) |
| `knowledge-memory/repo` | `knowledge-memory/rag` | Incremental Indexing (LightRAG, RAG Anything) |
| `session-memory` | `knowledge-memory/wiki` | **Nicht direkt** — nur via `longterm-memory` |

---

## 3. Folder Structure (vollständig)

```
brain-memory/
├── user-memory/
│   ├── index.md
│   ├── profile-xs.md / -s.md / -m.md / -l.md / -xl.md
│   └── topics/
│       ├── preferences.md
│       ├── projects.md
│       └── history.md
│
├── agent-memory/
│   ├── index.md
│   ├── _all/                    ← für alle Agenten gültig
│   │   ├── skills-xs.md … -xl.md
│   │   └── mcps.md
│   └── [AgentName]/             ← agenten-spezifisch
│       └── [AgentName]-xs.md … -xl.md
│
├── session-memory/
│   ├── index.md                 ← Übersicht aller Sessions
│   └── [SessionName-or-ID]/     ← z.B. "2025-06-29_gortex-setup"
│       ├── raw.md               ← Vollständige Session (Markdown-Export)
│       ├── tree-index.json      ← PageIndex Tree-Index
│       ├── summary-xs.md … -xl.md
│       ├── workedOn.md
│       ├── usedTools.md
│       ├── fails.md
│       ├── learnings.md
│       ├── learned.md
│       ├── agent-o.md           ← Optimierungs-Ideen für agent-o
│       ├── skills.md            ← Skill-Kandidaten
│       └── human-ai.md          ← Human-AI Connection Insights
│
├── project-memory/
│   ├── index.md
│   └── [ProjectName]/
│       ├── goals.md
│       ├── stack.md
│       ├── conventions.md
│       ├── todos.md
│       ├── adrs/
│       └── [ProjectName]-xs.md … -xl.md
│
├── knowledge-memory/
│   ├── repo/
│   │   └── [RepoName]/
│   │       ├── index.json       ← Gortex Tree-Index
│   │       ├── [RepoName]-xs.md … -xl.md
│   │       └── raw/             ← Original Files (optional, für Reference)
│   ├── wiki/
│   │   ├── coding/
│   │   │   └── swift/
│   │   │       ├── index.md
│   │   │       ├── log.md
│   │   │       ├── CLAUDE.md    ← Schema für LLM-Wiki (Karpathy Pattern)
│   │   │       ├── summary-xs.md … -xl.md
│   │   │       └── [topic].md
│   │   └── food/
│   └── rag/
│       ├── coding/
│       ├── swift/
│       └── food/
│
├── longterm-memory/             ← NEU: Promovierte Session-Insights
│   ├── index.md
│   ├── insights/
│   │   ├── [Topic]/
│   │   │   ├── insight-xs.md … -xl.md
│   │   │   └── source-session.md  ← Referenz zur Original-Session
│   └── patterns/                ← Wiederkehrende Muster, Best Practices
│
└── sensitivity-layer/           ← Separater Layer für Sensitivity-Tags
    ├── index.db                 ← SQLite: path → tag mapping (Runtime-Enforcement)
    └── config.yaml              ← Feature-Flags, Default-Policies
```

---

## 4. Tagging & Sensitivity Schema

### Sensitivity Tags (4 Stufen)

| Tag | Bedeutung | LLM-Zugriff | Logging |
|-----|-----------|-------------|---------|
| `[PUBLIC]` | Frei weiterleitbar, auch an externe Modelle | ✅ Alle | ✅ Voll |
| `[INTERNAL]` | Nur lokale Modelle, kein Cloud-LLM | ✅ Local Only | ✅ Metadaten |
| `[SENSITIVE]` | Niemals in LLM-Kontext, nur strukturelle Referenz | ❌ Nie | ❌ Kein Payload |
| `[PRIVATE]` | Absolut lokal, kein Logging | ❌ Nie | ❌ Nichts |

### Speicherung & Enforcement (entschieden)

- **Frontmatter** in JEDEM Memory-File (YAML: `sensitivity: PUBLIC`)
- **Index-DB** (`sensitivity-layer/index.db`) für Runtime-Checks (schnell)
- **Enforcement** via `brain_manager` Skill: vor LLM-Call → prüft Tags → filtert/redacted
- **Opt-In Only** — Layer wird **nur auf Anfrage** an Agent/LLM mitgegeben
- **Feature-Flag** — komplett abschaltbar (`sensitivity-layer/config.yaml`)

### Topic-Tags (zusätzlich)
- `[SKILL]`, `[MCP]`, `[CLI]`, `[FOR_ALL]`, `[FOR_AGENT:<name>]`, `[PROJECT:<name>]`

---

## 5. Doc-Size Tiering (xs–XL) + Token Budgets

| Size | Zweck | ~Token | Layer-Beispiele |
|------|-------|--------|-----------------|
| **xS** | System-Prompts, Referenz-Snippet | 50–150 | `profile-xs.md`, `skills-xs.md` |
| **S** | Quick-Context, Agent-Briefing | 150–500 | `summary-s.md`, `workedOn.md` |
| **M** | Normaler Kontext, Standard-Tasks | 500–2.000 | `summary-m.md`, `conventions.md` |
| **L** | Detaillierter Kontext, komplexe Tasks | 2.000–8.000 | `summary-l.md`, `adrs/` |
| **XL** | Vollständige Dokumentation, Onboarding | 8.000–32.000 | `summary-xl.md`, `raw.md` |

**Budget pro Agent/Task:** Wird in `agent-memory/_all/budgets.yaml` definiert (später).

---

## 6. Pipeline: Ingestion → Cleaner → Tagger → Layer-Routing

```
~/files/raw/ (Input)
       │
       ▼
┌──────────────────┐
│ CLEANER (Skill)  │  → Encoding, Whitespace, Boilerplate, PII-Redaction
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ TAGGER (Skill)   │  → Frontmatter: sensitivity, topic-tags, layer-routing, doc-size
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ ROUTER (Skill)   │  → Verschiebt nach brain-memory/<layer>/...
└──────────────────┘
```

**Implementation (entschieden): Hybrid**
- **Logik** in Hermes `brain_manager` Skills (`cleaner`, `tagger`, `router`)
- **Trigger** via OS: Cron + inotify (systemd path unit) auf `~/files/raw/`
- **Langfuse-Integration** — Tracing aller Pipeline-Schritte (Optional, via Config)

---

## 7. Tools & Services Matrix (Stand: Tool 1 & 2 geklärt)

| Tool | Layer | Input | Output | Trigger | Status |
|------|-------|-------|--------|---------|--------|
| **Gortex** | `knowledge-memory/repo/` | Code-Files (Swift, Python, TS, etc.) | MCP Resources (Live) + Tree-Index (`index.json`) + **Markdown xs–XL (via Exporter)** | Post-Clone (sofort) + Cron täglich (inkrementell) + Wöchentlich/On-Demand (Full-Rebuild) | ✅ Geklärt |
| **PageIndex** | `session-memory/` → `longterm-memory/` | Raw Sessions (Markdown) + Wiki-Docs | Tree-Index (JSON) + Summaries xs–XL + Analytics-Files (workedOn, usedTools, fails, learnings, learned, agent-o, skills, human-ai) | Session-Ende (Auto) + On-Demand | 🔄 Teilweise geklärt |
| **LLM-Wiki** (Karpathy Pattern) | `knowledge-memory/wiki/` | Markdown-Wikis + Sources | Persistentes, cross-referenziertes Wiki (LLM-maintained) | Ingest (Human-gesteuert) + Lint (Cron) | ⏳ Offener Hintergrund |
| **Understand-Anything** | `knowledge-memory/repo/` & `wiki/` | Code + Docs | ADRs, Summaries, Dependency Graphs | On-Demand (Agent/CI) + PR-Hook | ⏳ Offen |
| **LightRAG** | `knowledge-memory/rag/` | Wiki + Repo-Docs | Graph-RAG Index + Query API | Incremental (Watcher) + Cron (Full-Rebuild) | ⏳ Offen |
| **RAG Anything** | `knowledge-memory/rag/` | Multi-Modal Files (PDF, Images, Tables, Code) | Multi-Modal RAG Index | On-Demand + Cron | ⏳ Offen |

### Gortex Details (entschieden)
- **Scope:** Alle Repos in `~/github/loadedrepos/`
- **Kein `.git` nötig** — parst Source-Files direkt (Tree-Sitter)
- **MCP Live Access** für Agents (Context-minimiert: "Gib mir `APIClient.request`" → Snippet)
- **Swift-Focus:** Tree-Sitter Swift + evtl. SourceKit-LSP für Type-Resolution
- **Markdown-Exporter** (Skill/CLI): `gortex-export --format=markdown --output=knowledge-memory/repo/<name>/`

### PageIndex Details (teilweise entschieden)
- **Input:** Sessions als **Markdown** (bessere Lesbarkeit, `#` Headings für Hierarchie)
- **Analytics-Files:** **LLM-generiert** (Prompt + Tree-Index als Context)
- **Zusatz-Fragen** (agent-o, Skills, Human-AI): **Dynamische Prompts**
- **Output-Ablage:** `session-memory/<SessionName>/` mit allen Files
- **Offen:** Promotion-Kriterien → `longterm-memory` (Score? Review? TTL?)

---

## 8. Agent Access Patterns (teilweise geklärt)

> Wie fragt ein Agent gezielt nur sein Subset ab?

**Read-Permissions (entschieden):**

| Layer | Read Access |
|-------|-------------|
| `user-memory` | **Nur** `brain-user-memory` + `brain-memory-manager` |
| `session-memory` | **Nur** `brain-session-memory` + `brain-memory-manager` |
| Alle anderen | **Alle Profile** (Read-Only) + `brain-memory-manager` |

**Zugriffs-Patterns:**

- **Namespace-Prefix:** `agent-memory/_all/`, `agent-memory/<AgentName>/`
- **Tag-Filter:** `[FOR_AGENT:system_developer]` + `[FOR_ALL]`
- **MCP-Resources:** `gortex://`, `pageindex://`, `lightrag://` (Live, Token-minimal)
- **Doc-Size Budget:** Agent lädt max. X Tokens pro Layer (xs→S→M... bis Budget) — **Tracking pro Call** (Model, Provider, Thinking, Input/Output Tokens, Cost, Layer, Doc-Sizes) → Langfuse + JSONL
- **Layer-Selection:** Agent deklariert benötigte Layer → `brain-memory-manager` liefert Subset

**Offen:** Budget-Details pro Agent-Typ, Layer-Selection Konfiguration.

---

## 9. Sync / Replication / Retention / TTL Rules (offen)

| Layer | Retention | Sync | TTL |
|-------|-----------|------|-----|
| `session-memory` | Raw: unbegrenzt (Archiv) | PageIndex nach Session-Ende | — |
| `longterm-memory` | Unbegrenzt (kuratiert) | Manual Promotion | — |
| `knowledge-memory` | Unbegrenzt | Incremental (Watcher) | — |
| `user/agent-memory` | Unbegrenzt | On-Change | — |

**Offen:** `session-memory` Raw-Files — Compress nach X Tagen? TTL für Analytics?

---

## 10. Governance (teilweise geklärt)

**Architektur: Layer = Profil**

| Layer | Profil | Verantwortung |
|-------|--------|---------------|
| `user-memory` | `brain-user-memory` | User-Profile, Preferences, History |
| `agent-memory` | `brain-agent-memory` | Skills, MCPs, CLIs, Agent-Configs |
| `session-memory` | `brain-session-memory` | Raw Sessions, PageIndex, Analytics |
| `project-memory` | `brain-project-memory` | Goals, Stack, ADRs, TODOs |
| `knowledge-memory` | `brain-knowledge-memory` | Repos, Wiki, RAG |
| `longterm-memory` | `brain-longterm-memory` | Promovierte Insights, Patterns |
| `sensitivity-layer` | `brain-sensitivity` | Tags, Enforcement, Policies |
| **Orchestrator** | **`brain-memory-manager`** | Delegation, Audit-Log, Cross-Layer-Ops |

**Write-Permissions (entschieden):** Jedes Profil schreibt **nur** in seinen Layer. `brain-memory-manager` orchestriert via `delegate_task` und merged Git-Branches.

**Audit-Log:** `brain-memory-manager` loggt jeden Ingest/Promotion in `AUDIT.log` (wer, was, wo, wann, welcher Branch, Merge-Commit).

**Cross-Layer Reads:** Alle Profile dürfen alle Layer lesen (außer `user-memory` + `session-memory` → nur Owner + Manager).

**Index-Strategie:** Zentraler Index (`brain-memory/index.md`, Manager) + Profil-spezifische Indizes (`brain-memory/<layer>/index.md`, jeweiliges Profil).

**Promotion Flow:** `brain-session-memory` detected promotion-worthy → `delegate_task` → `brain-longterm-memory` + `brain-memory-manager` loggt.

**Git Workflow:** Jedes Profil arbeitet auf eigenem Branch `ingest/<layer>-<timestamp>` → `brain-memory-manager` merged `--no-ff` → `main` + Tag.

**Skills Location (Empfehlung: Hybrid):** Core-Skills (`cleaner`, `tagger`, `router`, `pageindex-runner`) in **Shared Library** (`~/system/clis/brain-memory-skills/`) — Profile importieren via `delegate_task` + `toolsets`.

**Backup:** Restic/Borg (täglich) + rclone zu Google Drive (5TB, Interim) → später Syncthing zu Home NAS.

**Migration:** Versionierte Migrations-Skills (`brain_manager migrate --from=v1 --to=v2`) — Git-Commit pro Migration.

**Backup-Verification:** Monatlich `restic check --read-data-subset=5%` + `rclone check` — Alert bei Fehler.

---

## Appendix: Examples

### Frontmatter (Standard)
```yaml
---
title: "Gortex Setup Session"
date: 2025-06-29
session_id: "2025-06-29_gortex-setup"
layer: "session-memory"
sensitivity: "INTERNAL"
tags: [SKILL, MCP, FOR_AGENT:system_developer, PROJECT:brain-memory]
doc_size: "M"
source: "hermes-export"
---
```

### Index-File Pattern (`index.md`)
```markdown
# Session Memory Index

| Session | Date | Topic | Size | Tags | Promoted |
|---------|------|-------|------|------|----------|
| 2025-06-29_gortex-setup | 2025-06-29 | Gortex MCP Config | M | [SKILL][MCP] | ❌ |
```

### Gortex MCP Config (Hermes)
```yaml
# ~/.hermes/config.yaml
mcp_servers:
  gortex:
    command: "gortex"
    args: ["mcp", "--index-dir", "~/brain-memory/knowledge-memory/repo"]
    env:
      GORTEX_MODEL: "gpt-4o-mini"
```

---

## Offene Fragen (Checklist)

- [ ] **Agent Access Patterns** — Budget-Details pro Agent-Typ, Layer-Selection Konfiguration
- [ ] **Sync/Retention/TTL** Details pro Layer (Compression, TTL für Analytics)
- [ ] **LLM-Wiki (Karpathy)** Integration & Schema-Design
- [ ] **Understand-Anything / LightRAG / RAG Anything** Rollen & Trigger
- [ ] **Doc-Size Budgets** pro Agent/Task
- [ ] **Langfuse Config** (Proxy vs. OTLP, Privacy-Settings pro Sensitivity-Tag)
- [ ] **Core-Skills Library Design** — Interface, Shared vs. Profil-spezifisch, Versioning
- [ ] **Plug&Play Package** Machbarkeit (später, via researcher)
- [ ] **Migration & Versioning Strategy** — Schema-Changes, Git-Tags, Upgrade-Pfad
- [ ] **Telemetry Config Details** — Langfuse OTLP, Privacy-Settings, Token-Tracking JSONL Schema

---

## Entscheidungs-Log (Auszug)

| Thema | Entscheidung | Begründung |
|-------|--------------|------------|
| Layer Count | 6 Layer + sensitivity-layer | `longterm-memory` neu für promotete Insights |
| Sensitivity Storage | Frontmatter + SQLite Index-DB | Human-readable + Runtime-Performance |
| Pipeline | Hybrid: Skills + OS-Triggers + Langfuse | Trennung Logik/Orchestrierung, Observability |
| Gortex Output | MCP Live + Tree-Index JSON + Markdown Export | Agents (Token-sparend) + Humans (Überblick) |
| PageIndex Input | Markdown Sessions | Lesbarkeit, `#` Headings für Hierarchie |
| PageIndex Analytics | LLM-generiert | Kontext-verstehend, flexibel |
| Session Output Path | `session-memory/<SessionName>/` | Human-lesbar, gruppiert |
| Promotion Flow | Hybrid: Score-Vorschlag + Human-Confirm, 28d Auto-Promote | Balance Autonomie/Kontrolle, kein Datenverlust |
| Read Permissions | user/session: Owner+Manager only; Rest: All Read-Only | Privacy für sensible Layer |
| Index Strategy | Zentral (Manager) + Profil-spezifisch | Cross-Layer Overview + Layer-Detail |
| Git Workflow | Per-Layer Branches, Manager merged --no-ff | Atomare Ingests, volle History, Rollback |
| Skills Location | Hybrid: Core in Shared Lib, Business-Logic pro Profil | Keine Duplication, echte Isolation |
| Backup | Restic + rclone (GDrive interim) → Syncthing (NAS) | 3-2-1, Versioning, Dedup, Encryption |
| Telemetry | Langfuse OTLP + JSONL Token-Tracking | Free-model observability, cost tracking, evaluation ready |