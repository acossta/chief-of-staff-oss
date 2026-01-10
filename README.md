<p align="center">
  <img src="assets/logo.svg" width="120" height="120" alt="Chief of Staff Logo">
</p>

<h1 align="center">Chief of Staff</h1>

<p align="center">
  <strong>Your AI-powered executive assistant for Claude Code</strong>
</p>

<p align="center">
  <a href="https://github.com/acossta/chief-of-staff-oss/stargazers"><img src="https://img.shields.io/github/stars/acossta/chief-of-staff-oss?style=flat-square&color=D97757" alt="GitHub Stars"></a>
  <a href="https://github.com/acossta/chief-of-staff-oss/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License"></a>
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude_Code-compatible-D97757?style=flat-square" alt="Claude Code Compatible"></a>
</p>

<p align="center">
  <a href="#-features">Features</a> •
  <a href="#-skills">Skills</a> •
  <a href="#-integrations">Integrations</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-usage">Usage</a> •
  <a href="#-creating-skills">Creating Skills</a>
</p>

---

## ✨ Features

- **Executive Coaching** — Get founder coaching based on methodologies from world-class coaches
- **Automated Standups** — Generate daily updates by aggregating work across all your tools
- **Custom Skills** — Build your own Claude Code skills with the skill creator
- **Deep Integrations** — Connect to Notion, Gmail, Calendar, Slack, and more via MCP

---

## 🎯 Skills

### Founder Coach

Executive coaching for startup founders based on methodologies from world-class coaches:

| Coach | Specialty |
|-------|-----------|
| **Bill Campbell** | Team building, operational decisions, giving feedback |
| **Jerry Colonna** | Self-doubt, burnout, emotional challenges |
| **Marshall Goldsmith** | Leadership development, behavioral change |
| **Ben Horowitz** | Hard things, wartime CEO decisions |
| **Fred Kofman** | Conscious business, authentic communication |
| **Matt Mochary** | CEO coaching, meeting efficiency |

```
/coach
```

The coach asks powerful questions rather than giving advice, helping you find your own wisdom through guided reflection.

---

### 📋 Standup Update

Generate daily standup updates by aggregating your work across multiple sources:

- **Notion** — Completed tasks, weekly goals progress
- **Gmail** — Emails you sent
- **Slack** — Messages and conversations
- **Calendar** — Meetings attended

```
/standup-update
```

Outputs a clean markdown summary organized by accomplishments, meetings, communications, and today's priorities.

---

### 🛠️ Skill Creator

Meta-skill for creating new Claude Code skills. Guides you through:

1. Understanding the problem
2. Planning the skill structure
3. Writing SKILL.md with proper frontmatter
4. Adding scripts, references, and assets
5. Testing and iteration

```
/skill-creator
```

---

## 🔗 Integrations

Chief of Staff connects to your tools via [MCP](https://modelcontextprotocol.io) (Model Context Protocol):

| Integration | Capabilities |
|-------------|-------------|
| <img src="https://www.notion.so/images/favicon.ico" width="16" height="16"> **Notion** | Read/write pages, databases, search workspace |
| <img src="https://ssl.gstatic.com/ui/v1/icons/mail/rfr/gmail.ico" width="16" height="16"> **Gmail** | Read emails, send messages, manage labels |
| <img src="https://ssl.gstatic.com/docs/doclist/images/drive_2022q3_32dp.png" width="16" height="16"> **Google Drive** | Read/write files, search, manage sharing |
| <img src="https://calendar.google.com/googlecalendar/images/favicons_2020q4/calendar_31.ico" width="16" height="16"> **Google Calendar** | View events, create meetings, check availability |
| <img src="https://a.slack-edge.com/80588/marketing/img/meta/favicon-32.png" width="16" height="16"> **Slack** | Read/send messages, search, manage channels |
| <img src="https://www.redditstatic.com/desktop2x/img/favicon/favicon-32x32.png" width="16" height="16"> **Reddit** | Fetch posts and comments |

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/acossta/chief-of-staff-oss.git
cd chief-of-staff-oss
```

### 2. Copy skills to your Claude Code config

```bash
cp -r .claude/skills ~/.claude/
```

### 3. Configure MCP servers

Copy `.mcp.json` to your project or merge with existing config:

```bash
cp .mcp.json ~/your-project/
```

### 4. Authenticate MCP servers

Each MCP server requires authentication. Follow the prompts when you first use each integration.

---

## 📖 Usage

### Founder Coaching Session

When you're stuck on a decision, facing a tough conversation, or dealing with self-doubt:

```
/coach
```

The coach will ask:
- "What's on your mind?"
- "What's the real challenge here for you?"
- "And what else?"

### Generate Standup Update

Each morning, generate a summary of yesterday's work:

```
/standup-update
```

The skill fetches data from all connected sources, asks clarifying questions, and outputs a markdown file to `updates/YYYY-MM-DD-morning-update.md`.

### Create a New Skill

Build your own custom skill:

```
/skill-creator
```

---

## 🏗️ Creating Skills

Skills follow a simple structure:

```
.claude/skills/my-skill/
├── SKILL.md           # Required: Skill definition with frontmatter
├── scripts/           # Optional: Helper scripts
├── references/        # Optional: Reference documents
└── assets/            # Optional: Images, data files
```

### SKILL.md Format

```markdown
---
name: my-skill
description: What the skill does
---

# My Skill

Instructions for Claude on how to execute this skill...
```

### Design Principles

| Principle | Description |
|-----------|-------------|
| **Single Responsibility** | One skill, one job |
| **Progressive Disclosure** | Simple surface, depth available |
| **Composability** | Skills can invoke other skills |
| **Fail Gracefully** | Handle missing data, timeouts |

---

## 📁 Project Structure

```
chief-of-staff/
├── .claude/
│   └── skills/
│       ├── founder-coach/     # Executive coaching skill
│       ├── skill-creator/     # Skill creation guide
│       └── standup-update/    # Daily standup generator
├── .mcp.json                  # MCP server configurations
└── README.md
```

---

## 🤝 Contributing

Contributions welcome! To add a new skill:

1. Fork the repository
2. Create your skill in `.claude/skills/your-skill/`
3. Add a `SKILL.md` with proper frontmatter
4. Submit a pull request

---

## 📄 License

MIT

---

