# 🤖 Agent Skills Collection

> Like [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills), but cooler. Because emojis. 😎

My curated collection of AI agent skills that extend ~~Claude Code~~ "Your Agentic Agent"'s capabilities with packaged instructions and scripts.

## 🌟 Available Skills

### 🔍 agent-feedback

**Analyze other agents' sessions and construct targeted corrective prompts to fix mistakes, correct context drift, or drive home task requirements.**

Ever watched a Claude Code agent go off the rails? This skill helps you:
- 📊 Discover and analyze session transcripts
- 🎯 Identify where agents went wrong
- 💬 Generate precise corrective prompts
- 🛡️ Check CLAUDE.md/AGENTS.md compliance
- 🔬 Debug agent decision-making

Perfect for when you need to understand why an agent did what it did, or when you need to get a derailed session back on track.

[📖 Read the full skill documentation](./agent-feedback/SKILL.md)

## 🚀 Installation

### Using the skills CLI

```bash
npx skills add Xalior/agent-skills
```

### Install a specific skill with the CLI

```bash
npx skills add Xalior/agent-skills --skill agent-feedback
```

### Manual Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/Xalior/agent-skills.git
   ```

2. Add the skill to your Claude Code configuration:
   ```bash
   cd agent-skills
   # Follow the Agent Skills format specification
   ```

## 📚 What are Agent Skills?

Agent Skills are packaged instructions that extend AI agent capabilities. Each skill contains:
- **SKILL.md** - Instructions that agents can execute
- **LICENSE** - Usage terms
- Optional **scripts/** and **references/** directories

## 🛠️ Skill Structure

```
agent-skills/
├── README.md
├── LICENSE
└── agent-feedback/
    ├── SKILL.md       # Agent instructions
    └── LICENSE        # MIT License
```
## 🤝 Contributing

Got a cool skill idea? Contributions welcome! Each skill should:
- ✅ Have a clear, specific purpose
- 📝 Include comprehensive SKILL.md documentation
- ⚖️ Include an MIT License
- 🎯 Follow the Agent Skills format specification

## 📋 Skill Format

Each skill should include frontmatter:

```yaml
---
name: skill-name
description: What the skill does
license: MIT
---
```

## 📜 License

MIT © 2026

---

Made with 💜 by someone cooler than Vercel (kidding, love them really)

*P.S. - If you're reading this and work at Vercel, this is all in good fun. Your agent-skills repo is genuinely awesome and inspired this one!* 🙏
