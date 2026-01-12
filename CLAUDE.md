# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Personal knowledge base for programming best practices, tools, and technical documentation. Content is bilingual (English primary, Chinese annotations).

## Repository Structure

```
docs/
├── languages/          # Programming languages (Rust, Python, Java, TS, Go)
│   └── _quickref/      # Language cheatsheets
├── tools/              # Dev tools, IDEs, package managers
│   └── _quickref/      # Tool command references
├── frameworks/         # Frameworks & libraries (Spring Boot, etc.)
│   └── _quickref/      # Framework annotations, configs
├── practices/          # Best practices, patterns, philosophy
│   └── _quickref/      # Checklists, quick guides
├── devops/             # Docker, K8s, CI/CD, monitoring
│   └── _quickref/      # DevOps commands
├── homelab/            # Self-hosting, networking, storage
│   └── _quickref/      # Homelab quick refs
├── ai/                 # LLMs, ML, prompt engineering
│   └── _quickref/      # AI tool references
└── resources/          # Learning resources, communities
```

## Content Guidelines

### Format
- Bilingual: English (primary) with Chinese annotations
- Use Markdown consistently
- `_quickref/` folders for cheatsheets and quick references
- Main folder level for deep-dive documentation

### Entry Pattern
```
**Tool/Technology Name** (optional URL)

- **指导建议**: [Chinese guidance text]
```

### Adding Content
1. Place in appropriate topic folder
2. Quick references go in `_quickref/` subfolder
3. Detailed guides go at folder root level
4. Include tool name in bold with optional URL
5. Provide practical guidance in Chinese under "指导建议"
6. Keep descriptions concise (1-3 sentences)

### Version Control
- Main branch: `main`
- Commits should clearly describe documentation changes
