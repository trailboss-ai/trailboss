<h1><img src="docs/brand/logo.svg" alt="Trailboss" height="64"></h1>

**Run your coding-agent herd: see every agent, what it costs, what it said, and bring it back.**

Trailboss works across Claude Code, Codex, opencode and pi, lists sessions from about 20 more agents, and takes any JSONL-writing agent through a config mapping. It's a headless core with a CLI whose `--json` output any frontend can use. Terminal apps plug in as *hosts*: [Herdr](https://herdr.dev) and tmux.

## What it does

- **Live fleet:** which agents are running where, grouped by folder, their status, which agent started which, and what needs you. Found from running processes, with no hooks in any agent's settings.
- **Notifications:** when an agent needs you or finishes a turn, on the desktop or in Herdr.
- **Resume:** bring sessions back after a reboot or Herdr restart, in the same folder with the same flags.
- **Session history and search:** full-text search across every agent's transcripts (optionally their thinking and tool output too), then resume any hit in place.
- **Usage, cost and insights:** tokens and API-equivalent $ by day, model, provider, project and session (cache read/write, tiered and fast-mode pricing included), next to Claude Code's own figure; streaks, a year heatmap, a model leaderboard.
- **Plans and quotas:** each subscription against what its usage would cost at API prices; 5-hour and weekly windows, reset times and pace for Claude, Codex and Cursor.
- **Delivery stats:** merged PRs, time to merge, AI-attributed share, and $ per merged PR, exact and estimated.
- **Which agent changed what:** every changed file attributed to the agent session that changed it, exactly or probably; review a session's changes and undo them.
- **In Herdr:** a status block, a dashboard, and opt-in agent-aware Files, editor and Changes panels ([below](#in-herdr)).

## Install

Linux for now (x86_64 and aarch64): live, resume and quotas rely on Linux (`/proc`, credential files).

**Release binary:** a static binary from the [latest release](https://github.com/trailboss-ai/trailboss/releases/latest), checked against its `SHA256SUMS`:

```sh
t=trailboss-$(uname -m)-unknown-linux-musl
curl -fsSLO https://github.com/trailboss-ai/trailboss/releases/latest/download/$t.tar.gz
curl -fsSL https://github.com/trailboss-ai/trailboss/releases/latest/download/SHA256SUMS | sha256sum -c --ignore-missing
tar xzf $t.tar.gz && install -m 755 $t/trailboss ~/.local/bin/
```

**With Herdr:** install the [Trailboss mod](https://github.com/trailboss-ai/trailboss-herdr) (through Herdr's package manager, or its `install.py`); its dashboard offers **Install** when the CLI is missing.

`stats` needs the GitHub CLI (`gh`), signed in.

## Usage

```sh
# Sessions
trailboss sessions                 # recent sessions, newest first
trailboss sessions --under ~/code --filter login   # also --agent codex,pi
trailboss search flaky login test  # full-text search, best matches first
trailboss search --all ENOENT      # also thinking and tool output (first 1 KB of each)
trailboss delete codex <id> --yes  # delete one session's transcript (shows what without --yes)

# Running agents
trailboss live                     # agents running now: where, status, cost so far, who started them
trailboss watch --desktop          # notify when an agent needs you or finishes a turn (30 s+)
trailboss events --json            # what changed since the last look, for scripts and frontends
trailboss track                    # record which session runs where (from a timer, outside Herdr)
trailboss resume --dry-run         # what a resume would type, and what it would skip and why
trailboss log                      # what resume did: resumed, failed, skipped, forgotten

# Cost and limits
trailboss usage --by model         # tokens and $; also --by day|project|session|all, --since YYYY-MM-DD
trailboss insights --period month  # streaks, heatmap, top models, leaderboard vs the previous period
trailboss plans --add "Claude Max" --price 200 --agents claude --renews 15
trailboss plans                    # each plan: API-equivalent cost this cycle, ×price, projected, last cycle
trailboss quota                    # plan limits: % used per 5h / weekly window, reset time, pace
trailboss stats                    # merged PRs, time to merge, AI share, $ per PR (--prs: each one)
trailboss prices                   # where prices come from; --refresh downloads LiteLLM's current table

# Files agents changed
trailboss changes --worktree .     # each changed file in this repo: which agent, exactly or probably
trailboss changes --under ~/code/api --since 2026-10-01   # agents' file edits, latest per file
trailboss review --under .         # sessions that changed files here; then:
trailboss review claude:<id>       # its files; --diff PATH; --undo (a plan; --yes to do it)

# Spaces in folders (for Herdr's sidebar, or any frontend)
trailboss folders                  # the folder tree; add NAME [--parent ID], edit, move, remove
trailboss folders edit f1 --rule billing --rule '~/code/client-*' --rule 'machine:vm-*'
trailboss folders file <space> f1  # put a space in a folder by hand (--top: at the top level)

# Setup
trailboss scan                     # what each source found
trailboss config                   # show ~/.config/trailboss/config.json; --set '<json>' to replace it
trailboss --json sessions          # any command as JSON, for scripts and the Herdr mod
```

Sessions are read from Claude Code (`~/.claude/projects`), Codex (`~/.codex/sessions`, `archived_sessions`), opencode (`~/.local/share/opencode/opencode.db`, read-only) and pi (`~/.pi/agent/sessions`). Trailboss never writes to agents' folders, with one exception you name and confirm: `delete`.

Every command first indexes whatever's new since the last run, into `~/.local/share/trailboss/index.db`, a private folder. The index holds transcript text, so it stays on your machine. It's only a cache: delete it and it's rebuilt. A new Trailboss version that changes its layout rebuilds it on the first run, which takes a few seconds. If you run Trailboss from more than one install (say Herdr's and your shell's), keep them on the same version, or each rebuilds the other's index.

### Live and resume (Linux)

`live` finds running agents from the process table, with no hooks in any agent. It shows each one's status: Claude Code's own idle/busy, Herdr's working/idle/done, or `suspended` for a Ctrl+Z'd agent. It also shows the session's title and cost so far. Agents in [Herdr](https://herdr.dev) terminals and tmux panes are listed by terminal (`host` says which); others by process. An agent started by another gets `parent`, the starter's `where`: from the host when it says (a Herdr sub-agent's terminal), else from the process tree (an agent run as a command by another).

`live`, `track` and `resume` remember which session ran where, in `tracked.json` next to the index. This file can't be rebuilt, so keep it. After a reboot:

- **In Herdr**, terminals come back as bare shells. `trailboss resume` types each lost session's resume command into its own terminal, in its folder, with the launch flags worth keeping (an allowlist per agent, never prompts or secrets). Then it checks that each agent came up. `--auto` resumes only what a reboot lost, after waiting for restored shells; after a Herdr restart alone, run it by hand. A parent agent whose sub-agents were resumed is told so.
- **In tmux** with [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect), panes come back as shells at the same `session:window.pane`, and `trailboss resume` types into them the same way. Without it the panes are gone, so it prints the commands (it only uses tmux's default server).
- **Elsewhere**, there's no terminal to type into, so `trailboss resume` prints the commands to run. A session you then resume by hand somewhere else is dropped from the lost list.

Tracking has to happen *before* the reboot. In Herdr the mod does it (and runs `resume --auto` when Herdr starts). Elsewhere, run `trailboss track` every minute or so from a systemd user timer or cron. Sessions skipped come with a reason: open elsewhere, folder gone, no saved conversation, or a folder name that can't be typed safely. Every resume, failure, skip and forgotten session is logged to `resume.log.jsonl` next to `tracked.json`, rolling over at 256 KB; `trailboss log` shows the latest.

### Notifications

`trailboss events` compares the agents running now with the last look and reports what changed:

- **needs you:** an agent is waiting for you (Herdr's "blocked", or the agent's own word for it);
- **finished:** a turn of 30 s or more ended (working → idle);
- **ended:** the agent is gone;
- **same folder:** two or more agents are *working* at once in the same git worktree (or folder, outside git), where they can overwrite each other's changes. Announced once when it starts, or when another agent joins. Idle agents sharing a folder are normal, so `live` only marks them (`shares_with`).
- **stuck:** an agent has been working for 10 minutes without its transcript changing (`live` shows `stuck` and `quiet_min`). It may be waiting on a long command, so it's worded as "no progress", not as an error. Announced once per stall.

`trailboss watch --desktop` keeps looking, every 10 s by default, and sends desktop notifications through `notify-send`. Claude Code only reports idle or busy, so for its sessions outside Herdr, "finished" works and "needs you" can't be told apart from idle. In Herdr, the status block notifies about agents **outside** Herdr's terminals (VS Code, plain terminals). Herdr already notifies about its own, so to avoid doubles Trailboss doesn't; set shared state `trailboss.notify.herdr` to true to get them all. Run one watcher at a time: they share what was last seen.

### Quotas

`trailboss quota` shows how much of each plan window you've used (Claude's 5-hour and weekly, Codex's), when each resets, and the **pace**: where usage ends up by the reset at the rate so far, flagged ⚠ over 100%. Pace appears once a quarter of the window has passed; before that a single busy session skews it. Over 100%, `limit_in` estimates when you'll hit the limit at this rate. In Herdr, the status block warns once per window when that happens.

This is one of two commands that use the network (the other is the opt-in `trailboss prices --refresh`, under [Cost](#cost)). It calls each provider's own usage endpoint with the sign-in token its CLI already stored (`~/.claude/.credentials.json`, `~/.codex/auth.json`). The token goes only to that provider and is never logged or saved. An expired token is reported, not refreshed: open the agent once and run it again. If the live call fails, Codex falls back to the last limits it logged in a session, marked with their age.

### Delivery stats

`trailboss stats` reads your pull requests from GitHub through `gh`, so `gh` handles sign-in and Trailboss never sees a GitHub token. For the last 30 days (`--days`) it shows merged PRs, median time to merge, open PRs and the oldest one's age. It also shows two measures of AI involvement:

- **`ai_marked`**: PRs whose commits, description or branch name say an AI tool made them (`Co-Authored-By: Claude`, "Generated with Claude Code", a `claude/` branch). PR texts are read only for this and not kept.
- **`with_agent_sessions` and `cost_usd`**: PRs whose branch had agent work, and what it cost. Each session's folder is matched to its GitHub repo through its git remote (worktrees included), and each response to the branch it was made on. `cost_per_pr_usd` is the cost spread over those PRs.

Work done on `main` and moved to a branch only at push time can't be matched to a PR by branch, so the matched cost is a lower bound. The `estimated_*` fields add a guess for it, marked "est." in Herdr: a PR also gets the agent work in its repo on its base branch (usually `main`) from when your previous PR into that repo and base was opened until this one was, and at most 7 days back. Those stretches don't overlap, so no work counts for two PRs. Per PR, `estimated_cost_usd` is the exact cost plus that share; the summary's `estimated_cost_usd` and `estimated_cost_per_pr_usd` total and average it. The exact fields stay as they are. `--prs` lists each PR with its cost; the `--json` summary also carries those rows under `prs`, so one call gets both.

Releases (`develop` → `main`) aren't counted, since their work was counted when it merged into `develop`.

Merged PRs from past weeks don't change, so they're kept in `stats-cache.json` next to the index and only the current week is asked for again. The cache holds PR metadata only: repo, number, title, dates, branches and AI marks. Delete the file to fetch everything again. In Herdr, the Stats tab shows the last result straight away, marked with its age, while it asks GitHub.

### Sources and config

Trailboss reads Claude Code, Codex (including compressed `.jsonl.zst` rollouts), opencode and pi with full usage. It also lists sessions from about 20 more agents (Gemini, Cursor, Droid, Copilot, Kiro, Cline, …) from the start of each session file. `~/.config/trailboss/config.json` (shown and saved, after checking, with `trailboss config [--set '<json>']`) can:

- switch a source off or point it elsewhere: `"sources": {"pi": {"enabled": false}}`;
- add any agent that writes JSONL with token usage, by mapping its fields: `"custom_sources"` (see [custom.rs](https://github.com/trailboss-ai/trailboss-adapters/blob/main/src/sources/custom.rs));
- set the GitHub accounts and work branches `stats` counts: `"github": {"logins": [...], "work_branch": {"owner/repo": "develop"}}`.

### Which agent changed what

Trailboss indexes every file edit agents make with their own tools: Claude Code's Edit and Write, Codex's patches, opencode's edit and write. `trailboss changes` lists the latest per file, with the session it came from. `--worktree DIR` goes through every file git sees changed in that repository and says who changed it since the last commit:

- **exact:** an agent's edit tool touched it;
- **probably:** no tool call did (agents also change files from the shell), but the file changed while exactly one agent session was working in that folder. Always labelled as a guess;
- **unknown:** otherwise, with the number of candidate sessions when more than one was active.

### Review and undo by session

`trailboss review AGENT:ID` lists every file a session changed, with:
- what it was before: a backup, or "created by the session";
- whether it still differs;
- which other sessions changed it afterwards.

`--diff PATH` shows the change from before the session to now.

`--undo` puts the session's files back as they were before it (or one file, with `--path`). It shows the plan first; `--yes` does it.
- **Where "before" comes from:** Claude Code's own file history (`~/.claude/file-history/`). Codex and opencode keep none, so their sessions can be reviewed but not undone.
- **What it changes:** it restores modified files and moves files the session created to the trash.
- **What it skips:** files another session changed later (`--force` to include them, losing that work too), and anything inside an agent's own folder.
- **Undoing the undo:** what it replaces is kept in `~/.local/share/trailboss/undo/<time>/` first.

### Cost

`cost_usd` is what the usage would cost at **API list prices**: what you'd pay per token, not what a Pro/Max plan charges you.

- **Cache tiers:** cache reads, 5-minute cache writes and 1-hour cache writes are priced separately.
- **Long prompts:** a response whose prompt is over a model's long-context threshold is priced entirely at the higher rate.
- **Fast mode:** a Claude Code response recorded with `usage.speed: "fast"` is billed at the model's fast multiplier (LiteLLM's `provider_specific_entry.fast`, e.g. 2× for Opus 5.5; the `fast` field in a price file). A model with no multiplier is priced at its standard rates.
- **Days** are in your local time.
- **Prices** come from [LiteLLM's table](https://github.com/BerriAI/litellm), bundled in the binary.
- **Newer prices without a release:** `trailboss prices --refresh` downloads LiteLLM's current table (a plain GET to `raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json`, nothing sent but the request) into `~/.config/trailboss/prices-litellm.json`, which then replaces the bundled prices. It never runs on its own. `trailboss prices` lists the layers and their dates; delete that file to go back to the bundled table.
- **Overrides:** to change or add a model, put the same shape in `~/.config/trailboss/prices.json`, in $ per million tokens. It wins over both tables, and a refresh never touches it:

  ```json
  {"models": {"my-local-model": {"input": 0, "output": 0}}}
  ```

  When there's no price for a model, the agent's own cost figure is used if it records one (opencode and pi do). Otherwise the response is counted in `unpriced` and left out of `cost_usd`.

**Against your plan:** `trailboss plans` compares each subscription you add with what its agents' usage would have cost at API prices: so far this billing cycle (and as a multiple of the price), projected to the cycle's end, and last cycle. A plan covers the agents you list, since a subscription pays for, say, Claude Code but not an API key used from another agent. Its cycle renews on `--renews` (1–28), else the 1st. The Herdr dashboard's Usage tab shows the same table.

**For Claude Code, this is a floor.** Claude Code also makes calls it doesn't write into transcripts (compaction, away summaries, titles), so its own `/cost` figure runs a few percent higher (median ~4% in testing). `trailboss sessions` and `trailboss live` show Claude Code's own figure beside Trailboss's as `reported_cost_usd`: the total it records in its transcript (`cost-state` records), at its own prices. It's empty for other agents and for sessions Claude Code didn't record one in.

- **Resumed sessions:** Claude Code's running total isn't consistent across a resume; it sometimes restarts lower. Trailboss keeps the highest total the file records and never adds totals together, so a resume that restarts the count can make it an undercount.
- **Sub-agents** aren't added in: Claude Code's total for the main session is shown as is.

## In Herdr

The [Trailboss mod](https://github.com/trailboss-ai/trailboss-herdr) is a thin frontend over the CLI's JSON:

- **Status block:** agents running, working and waiting; today's cost; the Claude 5-hour window and its pace; a **Resume N** button when a terminal lost its agent. It tracks sessions, notifies about agents outside Herdr, and resumes what a reboot lost when Herdr starts.
- **Dashboard** (Alt+Shift+T): Live (agents by folder, sub-agents under the agent that started them), Sessions, Usage, Quota and Stats tabs, plus an Insights window (Alt+Shift+I).
- **Files, editor and Changes** (opt-in): each changed file badged with the agent that changed it; create, rename, move, drag and trash; a multi-tab editor with highlighting, word wrap and find and replace that never overwrites an agent's edit unseen; staging and discarding by hunk; review and undo by session.
- **Spaces with folders** (opt-in): Herdr's spaces list with folders up to four levels deep. Drag spaces in, or let rules file them by folder or machine; fold a folder and it still shows when something in it needs you; show one folder alone.
- It can take the places of the insights, session-history, stats and usage-bar mods in your layouts.

Install, options and details: [trailboss-herdr](https://github.com/trailboss-ai/trailboss-herdr).

## Shape

```
frontends   CLI (JSON out) │ Herdr mod │ your own, over --json
core        sources (adapters) → store (SQLite + FTS5) → usage (pricing, plans, quotas, insights, stats)
            fleet (live, events, resume) │ files (changes, review and undo)
hosts       list / get / send / read terminals (Herdr via hc, tmux)
```

Transcripts stay local: the index lives on your machine and is never sent anywhere.

## The parts

| Repository | What | License |
|---|---|---|
| **trailboss** (here) | Issues, discussions, releases and these docs | |
| [trailboss-herdr](https://github.com/trailboss-ai/trailboss-herdr) | The Herdr mod | MIT |
| [trailboss-adapters](https://github.com/trailboss-ai/trailboss-adapters) | Reading each agent's transcripts; the Herdr and tmux hosts | MIT |
| [trailboss-specification](https://github.com/trailboss-ai/trailboss-specification) | The `--json` output of every command, as JSON Schemas, and how it may change | MIT |

The CLI itself is built from a private core and free to use; see [License](#license).

## Issues and contributing

Report bugs and ask for features [here](https://github.com/trailboss-ai/trailboss/issues), wherever the problem seems to be: in the CLI, an adapter or the mod. Questions and ideas go in [Discussions](https://github.com/trailboss-ai/trailboss/discussions). Pull requests are welcome in the public repositories above; see [CONTRIBUTING.md](CONTRIBUTING.md). Security problems: [SECURITY.md](SECURITY.md).

What changed in each version: [Releases](https://github.com/trailboss-ai/trailboss/releases).

## Prior art

Built from the Herdr mods agent-resume-plus, insights, session-history, usage-bar and stats.

## License

- **The `trailboss` binary** is free to use. Its source isn't public; the license terms ship with each release. Versions 0.1.0 to 0.3.0 were released under the MIT License, and stay so.
- **The adapters, the specification and the Herdr mod** are MIT; see each repository.
- **The name and logo** are trademarks; see [TRADEMARKS.md](TRADEMARKS.md).
