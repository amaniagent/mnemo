# Sensitivity Layer Configuration

This file controls the sensitivity tagging system. All memory files should have a `sensitivity` field in their frontmatter.

## Sensitivity Tags

| Tag | Meaning | LLM Access | Logging |
|-----|---------|------------|---------|
| `PUBLIC` | Free to forward, external models OK | All | Full |
| `INTERNAL` | Local models only, no cloud LLM | Local only | Metadata |
| `SENSITIVE` | Never in LLM context, structural ref only | Never | No payload |
| `PRIVATE` | Absolutely local, no logging | Never | Nothing |

## Enforcement

- **Frontmatter**: Every memory file MUST include `sensitivity: TAG`
- **Runtime Index**: SQLite DB at `index.db` for fast lookups
- **Enforcement Mode**: `strict` (default) — blocks non-compliant content from LLM context

## Configuration

```yaml
# sensitivity-layer/config.yaml
enabled: true
default_tag: "INTERNAL"
enforcement: "strict"
feature_flags:
  pii_redaction: true
  tag_enforcement: true
  log_filtering: true
```

## Tag Enforcement Rules

1. Files without sensitivity tag → rejected
2. `SENSITIVE`/`PRIVATE` content → stripped before LLM calls
3. `INTERNAL` content → only sent to local models
4. `PUBLIC` content → no restrictions