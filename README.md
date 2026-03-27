# Chief-of-Staff Plugin Template

A template for building your own Claude Code personal assistant plugin. Includes a full-featured email orchestrator, travel agent, credit card tracker, and file renamer as working examples.

## Getting Started

1. Click **Use this template** on GitHub to create your own repo
2. Clone your new repo locally
3. Register it as a local marketplace in Claude Code:

```bash
/plugin marketplace add ~/GitHub/your-repo-name
```

4. Install the plugins you want:

```bash
/plugin install chief-of-staff@my-plugins
/plugin install travel-agent@my-plugins
/plugin install credit-card-benefits@my-plugins
/plugin install rename-agent@my-plugins
```

5. Customize the plugins, add your own agents, and make it yours.

> **Note**: The marketplace name (`my-plugins`) comes from the `name` field in `.claude-plugin/marketplace.json`. Change it to whatever you like.

### Building MCP Servers

If plugins include bundled MCP servers, run the setup script:

```bash
./setup.sh
```

---

## Included Plugins

### chief-of-staff

**The email orchestrator.** Self-learning email triage that classifies your inbox, suggests actions based on your patterns, and delegates to specialized sub-agents.

#### Why Chief-of-Staff?

- **Questions-first flow**: Collect ALL decisions up front, execute in bulk at the end (faster)
- **Learns your patterns**: Records your choices vs suggestions, improves accuracy over time
- **Routes intelligently**: Detects packages, newsletters, and action items, sends them to the right handler
- **Multiple modes**: Interview mode (voice-friendly), batch mode (visual HTML), digest mode (summaries)
- **Unified interface**: One plugin, many capabilities

#### Quick Start

```bash
# 1. Configure your email provider
/chief-of-staff:setup

# 2. Learn patterns from existing folders
/chief-of-staff:learn

# 3. Triage your inbox
/chief-of-staff:triage     # Interactive Q&A mode
# OR
/chief-of-staff:batch      # Visual HTML batch mode
```

#### Commands

| Command | Description |
|---------|-------------|
| `/chief-of-staff:setup` | Configure email provider (Fastmail, Gmail, Outlook) |
| `/chief-of-staff:daily` | Full daily orchestration routine |
| `/chief-of-staff:status` | Quick dashboard of inbox status |
| `/chief-of-staff:triage` | Interactive questions-first triage |
| `/chief-of-staff:batch` | Visual HTML batch interface |
| `/chief-of-staff:parcel` | Process shipping emails to Parcel app |
| `/chief-of-staff:reminders` | Create reminders from action items |
| `/chief-of-staff:unsubscribe` | Unsubscribe from newsletters |
| `/chief-of-staff:digest` | Summarize automated emails |
| `/chief-of-staff:learn` | Bootstrap or update filing rules |
| `/chief-of-staff:analyze` | Find patterns in Trash/Archive |
| `/chief-of-staff:optimize` | Deep folder analysis and suggestions |
| `/chief-of-staff:rules` | View/manage filing rules |

#### Triage Modes

| Mode | Best For | How It Works |
|------|----------|--------------|
| `/chief-of-staff:triage` | Mobile, voice, thorough review | One-by-one Q&A with structured options |
| `/chief-of-staff:batch` | Desktop, quick visual review | HTML interface, review all at once |
| `/chief-of-staff:digest` | Quick status | Summary of automated emails |

#### Interview Mode Flow

```
PHASE 1: COLLECT (rapid Q&A)
-> Answer questions for each email
-> No waiting between emails

PHASE 2: EXECUTE (bulk processing)
-> All actions run at once
-> Single API call per folder

PHASE 3: LEARN (improve suggestions)
-> Record decisions vs suggestions
-> Update confidence scores
```

#### Built-in Sub-Agents

| Agent | Purpose |
|-------|---------|
| `inbox-interviewer` | Interactive questions-first triage |
| `inbox-to-parcel` | Package tracking from shipping emails |
| `inbox-to-reminder` | Create reminders from action items |
| `newsletter-unsubscriber` | Unsubscribe from newsletters |
| `digest-generator` | Summarize automated emails |
| `organization-analyzer` | Analyze Trash/Archive patterns |
| `pattern-learner` | Bootstrap filing rules from folders |
| `folder-optimizer` | Suggest folder reorganization |
| `decision-learner` | Learn from triage decisions |
| `batch-html-generator` | Visual batch triage interface |
| `batch-processor` | Execute batch triage decisions |
| `imessage-assistant` | Read and send iMessages via CLI |

#### Email MCP Setup (Required)

**Chief-of-Staff does NOT bundle an email MCP server.** You must configure your email provider separately.

**Fastmail (Recommended)**

Deploy your own Fastmail MCP server using Cloudflare Workers:

**Repository:** [omarshahine/fastmail-mcp-remote](https://github.com/omarshahine/fastmail-mcp-remote)

```bash
# After deploying, add to Claude Code:
claude mcp add --transport http fastmail https://your-worker.workers.dev/mcp
```

**Gmail**

**Official Integration:** Google services are available as [official integrations in Claude](https://www.anthropic.com/integrations). Enable Google Drive, Docs, and Gmail directly in Claude's settings.

**MCP Server:** For Claude Code CLI usage, use the Smithery Gmail server:
- [smithery.ai/server/gmail](https://smithery.ai/server/gmail)

**Outlook / Microsoft 365**

**Official Integration:** Microsoft services are available as [official integrations in Claude](https://www.anthropic.com/integrations). Enable Outlook, OneDrive, and other M365 services directly in Claude's settings.

**MCP Server:** For Claude Code CLI usage, use the Smithery Outlook server:
- [smithery.ai/server/outlook](https://smithery.ai/server/outlook)

#### Provider-Agnostic Architecture

Chief-of-Staff agents are **email provider-agnostic**. The active email provider is configured in `settings.yaml`, and agents dynamically load the appropriate email tools via ToolSearch at runtime. Switch email providers by updating `settings.yaml`. No agent changes needed.

#### Extending with Private Agents

You can add your own private agents for custom workflows:

1. Create agent files in `plugins/chief-of-staff/agents/`
2. Add commands in `plugins/chief-of-staff/commands/`
3. Add skills with domain knowledge in `plugins/chief-of-staff/skills/`
4. Private agents can call any COS sub-agent via the Task tool

See CLAUDE.md for the full plugin development guide.

---

### travel-agent

Flight research and trip tracking using multiple data sources.

| Agent | Model | Description |
|-------|-------|-------------|
| `google-flights` | sonnet | Search Google Flights for airfare pricing |
| `ita-matrix` | sonnet | Advanced fare research with detailed pricing rules |
| `flighty` | haiku | Query Flighty app for flight tracking |
| `tripsy` | haiku | Query Tripsy app for trip planning |

**Requirements:** `pip install fast-flights` (google-flights), Flighty/Tripsy macOS apps (flighty/tripsy)

---

### apple-pim

Native macOS integration for Calendar, Reminders, Contacts, and Mail using Apple's EventKit, Contacts, and JXA frameworks.

**Source:** [omarshahine/Apple-PIM-Agent-Plugin](https://github.com/omarshahine/Apple-PIM-Agent-Plugin)

> This plugin is sourced from an external GitHub repo. See the link above for full documentation.

| Command | Description |
|---------|-------------|
| `/apple-pim:calendars` | Manage calendar events |
| `/apple-pim:reminders` | Manage reminders |
| `/apple-pim:contacts` | Manage contacts |
| `/apple-pim:mail` | Manage Mail.app messages |
| `/apple-pim:configure` | Interactive setup |

**Requirements:** macOS 13+, Swift 5.9+, Node.js 18+

---

### credit-card-benefits

Track and maximize premium credit card benefits with anniversary-aware checklists.

| Card | Annual Fee | Key Benefits |
|------|------------|--------------|
| Amex Platinum | $895 | Monthly Uber/streaming, quarterly dining, annual airline |
| Venture X | $395 | Annual travel credit, anniversary miles |
| Chase Sapphire Reserve | $795 | Travel credit, DoorDash, Instacart |
| Delta SkyMiles Reserve | $650 | Companion cert, Delta Stays, monthly credits |
| Alaska Airlines Atmos Summit | $395 | Companion fare, lounge passes |

| Command | Description |
|---------|-------------|
| `/credit-card-benefits:configure` | Set up cards and data sources |
| `/credit-card-benefits:sync` | Sync transactions |
| `/credit-card-benefits:status` | View all benefits |
| `/credit-card-benefits:remind` | Benefits expiring soon |

---

### rename-agent

AI-powered file renaming with pattern-based naming. Analyzes PDFs, images, and text files for smart classification.

```bash
/rename-agent:rename ~/Downloads/tax-docs
/rename-agent:rename ~/Documents/receipts --pattern "{Date:YYYY-MM-DD} - {Merchant}"
```

**Requirements:** Python 3.10+, `ANTHROPIC_API_KEY`, `pip install claude-rename-agent`

**Source:** [omarshahine/claude-rename-agent](https://github.com/omarshahine/claude-rename-agent)

---

## Creating New Plugins

1. Create directory: `plugins/my-plugin/`
2. Add manifest: `plugins/my-plugin/.claude-plugin/plugin.json`
3. Add agents, skills, or commands
4. Register in `.claude-plugin/marketplace.json`
5. Bump the marketplace version
6. Update this README

### Plugin Structure

```
plugins/
└── my-plugin/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── agents/
    │   └── my-agent.md
    ├── skills/
    │   └── my-skill/
    │       └── SKILL.md
    ├── commands/
    │   └── my-command.md
    └── README.md
```

### Marketplace Configuration

Add to `.claude-plugin/marketplace.json`:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "description": "Description of my plugin",
  "version": "1.0.0",
  "keywords": ["keyword1", "keyword2"],
  "category": "productivity"
}
```

---

## License

MIT
