# AI & Engineering Learning Notes

**English** | [中文](README_CN.md)

Personal learning journal on AI engineering — Claude Code context engineering, full-stack development, and technical writing.

[![AI Engineering](https://img.shields.io/badge/AI-Engineering_Notes-blue?style=for-the-badge)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

---

## Highlights

### Claude Code Context Engineering (`claude-code/`)

A **6-layer architecture** for building production-grade Claude Code workflows:

| Layer | File | What it covers |
|-------|------|----------------|
| Layer 1 | Long-term Context | CLAUDE.md, Memory, rules — the "personality" of your Claude |
| Layer 2 | Tool Capabilities | Scripts, MCP Servers, built-in tools — what Claude *can do* |
| Layer 3 | Skills (Workflows) | Slash commands, SOPs — *how* Claude does recurring tasks |
| Layer 4 | Hooks | Automatic enforcement — the "must do" layer |
| Layer 5 | Subagents | Parallel workers — scaling with isolation |
| Layer 6 | Verification Loop | Testing, validation — "done" has a checkable definition |

Plus 3 real-world case studies:
- **Writing Quality System** — docx format checking pipeline
- **Bid Document Full Process** — 12-chapter technical bid, multi-agent parallel writing
- **VPS Infrastructure** — server setup automation with Claude Code

### Other Topics

- `ai/` — AI learning path, LangChain, RAG patterns
- `fullstack/` — Full-stack development notes
- `writing/` — Technical report writing framework
- `leveling/` — Skill progression tracking

## Structure

```
Learn/
├── claude-code/          ← Claude Code 6-layer architecture + case studies
│   ├── 00-总览与主线地图.md
│   ├── Layer1~Layer6     ← One file per layer
│   ├── 实战-*.md          ← Real-world case studies
│   └── tw93/             ← Extended reference (12 chapters)
├── ai/                   ← AI/ML learning notes
├── fullstack/            ← Full-stack development
├── writing/              ← Technical writing framework
└── leveling/             ← Skill progression
```

## License

MIT
