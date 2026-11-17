# Onbo Hub Skills

Agent skills for [Onbo Hub](https://onbo-hub.com): the full onboarding of mobile apps with verified revenue — every screen from first launch to the paywall, labeled by screen type — plus how each app monetizes, the TikTok / Instagram posts that grew it, and its Meta / Apple ads.

Give your AI agent real examples from apps that make money while it designs your onboarding, or have it analyze how they monetize and grow.

## What your agent can do

- "Design a paywall for my meditation app based on what top Health & Fitness apps do."
- "Build a quiz-style onboarding like the top apps, and keep their screen order."
- "When in the flow do apps ask for an App Store rating?"
- "What share of $100k+ apps offer a free trial, and how do they show it?"
- "Which TikTok hooks grew these apps? Which Meta ads have been running the longest?"

The agent picks comparable apps, pulls the matching screens (paywall, question, analyzing, personalized plan, rating request, permission primer…), looks at the images and turns what they share into a plan for your app.

## MCP server

The easiest way to connect, especially from claude.ai or ChatGPT. Sign in with your Onbo Hub account (Pro plan) — no key to copy. The tools return the screen images themselves, so the AI can look at paywalls and onboarding screens directly.

```text
https://onbo-hub.com/mcp
```

- **claude.ai / Claude Desktop:** Settings → Connectors → Add custom connector, paste the URL, then sign in.
- **ChatGPT:** add a custom connector (MCP) with the URL, then sign in.
- **Claude Code:**

  ```bash
  claude mcp add --transport http onbo-hub https://onbo-hub.com/mcp
  ```

  Then run `/mcp` in Claude Code and sign in.
- **Cursor and other tools that send their own headers:** use your API key instead of signing in.

  ```json
  {
    "mcpServers": {
      "onbo-hub": {
        "url": "https://onbo-hub.com/mcp",
        "headers": { "Authorization": "Bearer ohub_your_key_here" }
      }
    }
  }
  ```

Tools: `find_screens` (one kind of screen across apps, e.g. every paywall, highest revenue first), `search_apps`, `get_app`, `view_screens`, `get_stats`.

## Install the skill

The skill calls the REST API with an API key. Use it if you prefer skills over MCP.

### Claude Code

```text
/plugin marketplace add onbo-hub/skills
/plugin install onboardhub-data@onbo-hub
```

Or as a personal skill:

```bash
mkdir -p ~/.claude/skills/onboardhub-data
curl -fsSL https://onbo-hub.com/skill.md -o ~/.claude/skills/onboardhub-data/SKILL.md
```

### Codex, Cursor and other agents

```bash
npx skills add onbo-hub/skills
```

Or point your agent at <https://onbo-hub.com/skill.md> — it is the same file as plain Markdown.

### claude.ai

```bash
mkdir onboardhub-data
curl -fsSL https://onbo-hub.com/skill.md -o onboardhub-data/SKILL.md
zip -r onboardhub-data.zip onboardhub-data
```

Upload `onboardhub-data.zip` under **Settings → Capabilities → Skills**. Code execution has to be allowed to reach `onbo-hub.com`.

## API key

The skill calls the Onbo Hub REST API, which needs a **Pro** plan.

1. Create a key at <https://onbo-hub.com/dashboard>.
2. Set it as an environment variable:

   ```bash
   export ONBOARDHUB_API_KEY=your_key
   ```

The dashboard can also give you a copy of SKILL.md with your key already in it. Keep that copy private and never commit it.

API reference: <https://onbo-hub.com/docs/api>

## Data terms

The data is for reference and internal use: to inform your own product, design and marketing decisions. Reselling or redistributing it, building it into other products, or using it to train AI models is not allowed without written permission. See the [Terms of Service](https://onbo-hub.com/tos#usage).

## About this repository

`skills/onboardhub-data/SKILL.md` is synced every day from <https://onbo-hub.com/skill.md>, so changes to the API show up here on their own. For problems with the skill, open an issue.

---

## 日本語

[Onbo Hub](https://onbo-hub.com) のデータを AI エージェントから使うためのスキルです。売上のあるアプリのオンボーディング（最初の画面から課金画面まで、画面の種類つき）・課金の仕組み・伸びた SNS 投稿・広告を、オンボーディングや課金画面を作りながら参考にしたり、分析したりできます。

- MCP サーバー（いちばん手軽）: `https://onbo-hub.com/mcp` を claude.ai・ChatGPT のカスタムコネクタに入れるか、Claude Code で `claude mcp add --transport http onbo-hub https://onbo-hub.com/mcp` → `/mcp` でログイン
- スキル（Claude Code）: `/plugin marketplace add onbo-hub/skills` → `/plugin install onboardhub-data@onbo-hub`
- そのほかのエージェント: `npx skills add onbo-hub/skills`、または <https://onbo-hub.com/skill.md> を読ませる
- API キーは Pro プランで [ダッシュボード](https://onbo-hub.com/dashboard) から発行できます
