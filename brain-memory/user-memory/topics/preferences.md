---
title: "User Preferences"
sensitivity: "INTERNAL"
tags: ["USER", "PREFERENCES", "M"]
doc_size: "M"
layer: "user-memory"
---

# User Preferences

## Communication Style
- **Language:** German (primary), English (technical terms)
- **Tone:** Direct, technical, concise
- **Detail Level:** High — prefers full context over summaries

## Tooling Preferences
- **Editor:** VS Code / Cursor
- **Terminal:** Hermes TUI / kitty
- **Shell:** zsh
- **Package Manager:** uv (Python), npm/pnpm (Node), brew (macOS)
- **Container Runtime:** Docker / Podman
- **Version Control:** Git (conventional commits, --no-ff merges)

## Development Workflow
- **Branching:** Feature branches per ingest/task
- **Commits:** Atomic, signed, conventional format
- **Reviews:** Self-review via subagents before PR
- **Testing:** TDD where feasible, integration tests for infra
- **Documentation:** Markdown-first, living docs in repo

## AI/LLM Preferences
- **Primary Models:** Free/OpenRouter models (Nemotron, Qwen, etc.)
- **Local Inference:** llama.cpp, vLLM, Ollama
- **Observability:** Langfuse (self-hosted), JSONL token tracking
- **Privacy:** Sensitivity tags enforced, no cloud for SENSITIVE/PRIVATE
- **Token Tracking:** Per-call, per-layer, per-model

## Infrastructure
- **Provisioning:** Ansible, Terraform/OpenTofu
- **Orchestration:** Docker Compose, systemd
- **Monitoring:** Prometheus, Grafana, Loki, Alertmanager
- **Backup:** Restic + rclone (GDrive), future Syncthing to NAS
- **VPN/Mesh:** Tailscale, WireGuard

## Knowledge Management
- **Primary:** Hermes brain-memory profiles (6 layers)
- **Secondary:** Obsidian vault (~/brains/)
- **Code Navigation:** Gortex MCP
- **Session Analytics:** PageIndex
- **RAG:** LightRAG, RAG Anything