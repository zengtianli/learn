# AI & Engineering Learning Notes

**English** | [中文](README_CN.md)

Personal learning journal on AI engineering — Claude Code context engineering, full-stack development, and technical writing.

[![AI Engineering](https://img.shields.io/badge/AI-Engineering_Notes-blue?style=for-the-badge)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

---

## Highlights

### Claude Code Context Engineering (`claude-code/`)

A **6-layer architecture** for building production-grade Claude Code workflows:

| Layer | What it covers |
|-------|----------------|
| Layer 1 | Long-term Context — CLAUDE.md, Memory, rules, token budget |
| Layer 2 | Tool Capabilities — scripts, MCP, naming conventions |
| Layer 3 | Skills (Workflows) — SOPs, descriptor optimization |
| Layer 4 | Hooks — deterministic enforcement |
| Layer 5 | Subagents — isolation > parallelism, permission control |
| Layer 6 | Verification Loop — L0-L5 layered validation |

Plus supplementary topics: Agent core loop, concept boundaries, prompt caching, high-frequency commands.

**3 real-world case studies** + audit reports with 6-dimension harness scoring.

### Other Topics

- `ai/` — AI learning path, LangChain, RAG patterns
- `writing/` — Technical report writing framework

## Structure

```
Learn/
├── claude-code/
│   ├── theory/       ← 6-layer architecture + supplementary topics (11 files)
│   ├── practice/     ← Real-world case studies & logs (4 files)
│   ├── strategy/     ← Tool strategies & workflow optimization
│   └── audit/        ← Health check reports & scoring reference
├── ai/               ← AI/ML learning notes
└── writing/          ← Technical writing framework
```

## License

MIT
