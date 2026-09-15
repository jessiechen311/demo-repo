# Claude Code — HelloFresh Canada starter setup

A quick-start guide for teammates who want Claude Code configured the way our team uses it (Bedrock-backed, with the Databricks MCP and a shared team context library).

## 1. Install Claude Code

- **VS Code extension** (recommended): search "Claude Code" in the Extensions marketplace, or grab it from claude.com/claude-code
- **CLI**: `npm install -g @anthropic-ai/claude-code` — then run `claude` in any project directory

## 2. AWS SSO — one-time prerequisite

Claude Code runs through **AWS Bedrock** on our account, not the public Anthropic API. You need `aws-cli` and an SSO profile.

```bash
# Install aws-cli if you don't have it
brew install awscli

# Configure the SSO profile
aws configure sso --profile sso-bedrock-user
```

Ask your team lead or platform contact for:
- SSO start URL
- SSO region
- The Bedrock role name you should assume

Verify it works:
```bash
aws sso login --profile sso-bedrock-user
aws bedrock list-inference-profiles --profile sso-bedrock-user --region eu-west-1 --max-results 5
```

## 3. Config files — `~/.claude/`

Create the directory and drop in these two files.

### `~/.claude/settings.json`

Start with [`settings.starter.json`](./settings.starter.json) in this folder. Copy it to `~/.claude/settings.json` and replace the two placeholders:
- `<YOUR_DATABRICKS_PAT>` — generate a Databricks Personal Access Token at https://hf-query-engine.cloud.databricks.com (User Settings → Developer → Access tokens). **Do not share this token with anyone.**
- `<YOUR_USERNAME>` — your macOS username (whatever `whoami` prints)

### `~/.claude/CLAUDE.md`

Your user-global instructions. Claude reads this at the start of every session. Start with [`CLAUDE.starter.md`](./CLAUDE.starter.md) and edit the name/role/team line to match you.

## 4. Team context library (optional but recommended)

The `@/Users/.../Context files/INDEX.md` import in `CLAUDE.md` pulls in a shared library of team briefs, metric definitions, and reusable skills. If you're on POA, ping Afagh for a copy of the current files. Other teams should build their own — the pattern is:

```
~/Claude/Projects/Context files/
├── INDEX.md                          # entry point (loaded every session)
├── team-brief-<your-team>.md         # who you are, what you own, cadence
├── metrics-and-sources-brief.md      # canonical metric definitions + source tables
└── SKILL_*.md                        # reusable prompt patterns
```

Keep `INDEX.md` short — it just points to the other files. Anything imported gets loaded into every conversation, so watch the token budget.

## 5. First run

```bash
cd ~/some-project
claude
```

Try `/help` for the built-in command list. If Bedrock auth fails, run `aws sso login --profile sso-bedrock-user` and try again — the `awsAuthRefresh` setting handles this automatically once configured.

## 6. Growing your setup over time

- **Permissions**: Claude will prompt you before running unfamiliar commands. Approve them once, and they'll be added to `settings.json` `permissions.allow` for the future. If prompts pile up, run `/fewer-permission-prompts` to batch-review.
- **Skills**: drop custom skills in `~/.claude/skills/<skill-name>/SKILL.md`. Invoke with `/<skill-name>`.
- **Project-level config**: any repo can have its own `.claude/CLAUDE.md` and `.claude/settings.json` for project-specific rules and permissions — these merge with your user-global config.

## Reference — what's in the starter settings

| Key | Why it's there |
|---|---|
| `awsAuthRefresh` | Auto-refreshes SSO creds when they expire |
| `env.CLAUDE_CODE_USE_BEDROCK` | Routes API calls through AWS Bedrock instead of api.anthropic.com |
| `env.ANTHROPIC_MODEL` | Which Bedrock model ID to use (Opus 4.7 in eu-west-1) |
| `mcpServers.databricks-sql` | Lets Claude run SQL against our Databricks warehouse |
| `permissions.allow` | Pre-approved commands Claude won't prompt for |
| `permissions.additionalDirectories` | Paths outside the project Claude is allowed to read |

## Troubleshooting

- **"Unable to locate credentials"** → run `aws sso login --profile sso-bedrock-user`
- **"Access denied to model"** → your IAM role doesn't have Bedrock invoke permission; ping platform team
- **Databricks MCP returns 401** → regenerate your PAT; tokens expire
- **Model not found** → check `ANTHROPIC_MODEL` matches an inference profile in your region (`aws bedrock list-inference-profiles`)

---

Questions? Ask Afagh.
