<h1><img src="docs/brand/logo.svg" alt="Trailboss" height="64"></h1>

**Run your coding-agent herd: see every agent, what it costs, what it said, and bring it back.**

Trailboss works across Claude Code, Codex, opencode, pi, Gemini CLI and Copilot CLI, lists sessions from about 20 more agents, and takes any JSONL-writing agent through a config mapping. It's a headless core with a CLI whose `--json` output any frontend can use. Terminal apps plug in as *hosts*: [Herdr](https://herdr.dev) and tmux (and tuios, experimental and untested).

## What it does

- **Live fleet:** which agents are running where, grouped by folder, their status, which agent started which, and what needs you. Found from running processes, with no hooks in any agent's settings.
- **Notifications:** when an agent needs you or finishes a turn, and why it needs you, on the desktop or in Herdr.
- **Send and keep going:** type a message into idle agents, queue one for an agent mid-turn, or keep a session going until a check like `cargo test` passes.
- **Tasks:** start an agent on a brief in its own git worktree and branch, see what it did, and merge it when you accept it.
- **Checkpoints:** save a session's folder as a git commit, by hand or after every turn, and put it back.
- **Resume:** bring sessions back after a reboot or Herdr restart, in the same folder with the same flags.
- **Session history and search:** full-text search across every agent's transcripts (optionally their thinking and tool output too), then resume any hit in place, or read it as a conversation.
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

`stats` needs the GitHub CLI (`gh`), signed in. `trailboss doctor` then says what's working and what isn't.

## Usage

```sh
# Sessions
trailboss sessions                 # recent sessions, newest first
trailboss sessions --under ~/code --filter login   # also --agent codex,pi
trailboss search flaky login test  # full-text search, best matches first
trailboss search --all ENOENT      # also thinking and tool output
trailboss thread claude:<id>       # one session as a conversation: each prompt, the reply, tokens, cost
trailboss delete codex <id> --yes  # delete one session's transcript (shows what without --yes)

# Running agents
trailboss live                     # agents running now: where, status, why it needs you, model, context, cost
trailboss live --pin t3            # keep an agent at the top (--unpin to let it go)
trailboss watch --desktop          # notify when an agent needs you or finishes a turn (30 s+)
trailboss events --json            # what changed since the last look, for scripts and frontends
trailboss track                    # record which session runs where (from a timer, outside Herdr)
trailboss resume --dry-run         # what a resume would type, and what it would skip and why
trailboss log                      # what resume did: resumed, failed, skipped, forgotten

# Talking to agents
trailboss send --to t3 "rebase on main"   # who would get it; --yes types it (--to all, repeatable)
trailboss send --to all --queue --yes "stop after this step"   # agents mid-turn get it when the turn ends
trailboss queue                    # queued messages and the last ones typed; queue rm q4 (or --all)
trailboss until t3 -- cargo test   # after each turn run the check; type the failure in until it passes
trailboss until ls                 # loops running now; until pause|resume|stop <id|session|all>

# Tasks: a brief, an agent, its own worktree
trailboss task add "Fix the login redirect. Done when: ..." --agent codex   # the plan; --yes starts it
trailboss task ls                  # each task: to do, working, needs you, ready, done
trailboss task show 3              # what it did: last message, commits, files, checks; then:
trailboss task accept 3 --yes      # fast-forward your branch to it (task rm 3 --yes: drop it)
trailboss agents                   # the definitions task add --agent <name> starts; agents show NAME

# Checkpoints
trailboss checkpoint save claude:<id>     # the session's folder as a git commit; list, diff N
trailboss checkpoint restore claude:<id> 2   # what it would write and remove; --yes does it

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
trailboss doctor                   # what's working and what isn't, each with the command that fixes it
trailboss version                  # the version, and the --json api version frontends check
trailboss scan                     # what each source found
trailboss config                   # show ~/.config/trailboss/config.json; --set '<json>' to replace it
trailboss --json sessions          # any command as JSON, for scripts and the Herdr mod
```

Sessions are read from Claude Code (`~/.claude/projects`), Codex (`~/.codex/sessions`, `archived_sessions`), opencode (`~/.local/share/opencode/opencode.db`, read-only), pi (`~/.pi/agent/sessions`), Gemini CLI (`~/.gemini/tmp`) and Copilot CLI (`~/.copilot/session-state`). Trailboss never writes to agents' folders, with one exception you name and confirm: `delete`.

`trailboss thread AGENT:ID` shows one session as a conversation: a turn per prompt, with its reply, model, tokens, cost, time and how many tool results it had. Slash commands are left out; a background task's report is marked `"from": "agent"`.

Every command first indexes whatever's new since the last run, into `~/.local/share/trailboss/index.db`, a private folder. The index holds transcript text, so it stays on your machine. It's only a cache: delete it and it's rebuilt. A new Trailboss version that changes its layout rebuilds it on the first run, which takes a few seconds. If you run Trailboss from more than one install (say Herdr's and your shell's), keep them on the same version, or each rebuilds the other's index.

### Live and resume (Linux)

`live` finds running agents from the process table, with no hooks in any agent. It shows each one's status: Claude Code's own idle/busy, Herdr's working/idle/done, or `suspended` for a Ctrl+Z'd agent. It also shows the session's title and cost so far. Agents in [Herdr](https://herdr.dev) terminals and tmux panes are listed by terminal (`host` says which); others by process. An agent started by another gets `parent`, the starter's `where`.

Each agent also gets, where its transcript or settings say (null where they don't):

- **Its turn:** `turn` and `turn_ended_ms`, and `status_since` (shown as `for`, "12m").
- **Why it needs you:** `needs`, also a column: "Plan to approve", a question's first words, the command or edit it asks permission for ("Run: npm test", "Edit src/main.rs"), "Rate limited", or the error its turn ended on. `needs_kind` says which kind (`run`, `edit`, `plan`, `question`, `reply`, …).
- **Model and context:** `model` from its last response, `effort` where the agent records one, and how full its context window is (`context_tokens`, `context_window`, `context_pct`).
- **Load:** `cpu_pct`, `rss_bytes` and `procs` over its whole process tree (the `cpu` and `mem` columns). An agent making no progress for a while is `stuck` only when it isn't busy either; a busy one (a long build, a test run) is `quiet_busy`.
- **Background jobs** it left running (`background`, the `jobs` column): Claude Code's background shells and monitors, Codex's processes still alive after their turn. Only listed; nothing is stopped.

The table lists agents that need you first, then done, working and the rest, sub-agents under the agent that started them. `trailboss live --pin <terminal|session>` keeps one at the top (`pinned`, kept in `pins.json` next to the index); `--unpin` lets it go. The `--json` order doesn't change.

**Gemini CLI and Copilot CLI** agents show up too, with their session, status, turn and what they need. A running Copilot session writes its tokens only when it ends, so `live` shows them as null with `tokens_note: "not until it ends"`, not 0. Copilot CLI support in `live` and `send` is new and largely untested: treat it as experimental.

`live`, `track` and `resume` remember which session ran where, in `tracked.json` next to the index. This file can't be rebuilt, so keep it. After a reboot:

- **In Herdr**, terminals come back as bare shells. `trailboss resume` types each lost session's resume command into its own terminal, in its folder, with the launch flags worth keeping (an allowlist per agent, never prompts or secrets). Then it checks that each agent came up. `--auto` resumes only what a reboot lost, after waiting for restored shells; after a Herdr restart alone, run it by hand. A parent agent whose sub-agents were resumed is told so.
- **In tmux** with [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect), panes come back as shells at the same `session:window.pane`, and `trailboss resume` types into them the same way. Without it the panes are gone, so it prints the commands (it only uses tmux's default server).
- **Elsewhere**, there's no terminal to type into, so `trailboss resume` prints the commands to run. A session you then resume by hand somewhere else is dropped from the lost list.

Tracking has to happen *before* the reboot. In Herdr the mod does it (and runs `resume --auto` when Herdr starts). Elsewhere, run `trailboss track` every minute or so from a systemd user timer or cron. Sessions skipped come with a reason: open elsewhere, folder gone, no saved conversation, or a folder name that can't be typed safely. Every resume, failure, skip and forgotten session is logged to `resume.log.jsonl` next to `tracked.json`, rolling over at 256 KB; `trailboss log` shows the latest.

### Notifications

`trailboss events` compares the agents running now with the last look and reports what changed:

- **needs you:** an agent is waiting for you (Herdr's "blocked", or the agent's own word for it), with the reason `live` shows;
- **finished:** a turn of 30 s or more ended (working → idle). `--json` also reports every finished turn, however short, as `turn_finished`;
- **ended:** the agent is gone;
- **same folder:** two or more agents are *working* at once in the same git worktree (or folder, outside git), where they can overwrite each other's changes. Announced once when it starts, or when another agent joins. Idle agents sharing a folder are normal, so `live` only marks them (`shares_with`).
- **stuck:** an agent has been working for a while without making progress and isn't busy (`live` shows `stuck` and `quiet_min`). It may be waiting on a long command, so it's worded as "no progress", not as an error. Announced once per stall.

`trailboss watch --desktop` keeps looking, every 10 s by default, and sends desktop notifications through `notify-send`. Claude Code only reports idle or busy, so for its sessions outside Herdr, "finished" works and "needs you" can't be told apart from idle. In Herdr, the status block notifies about agents **outside** Herdr's terminals (VS Code, plain terminals). Herdr already notifies about its own, so to avoid doubles Trailboss doesn't; set shared state `trailboss.notify.herdr` to true to get them all. Run one watcher at a time: they share what was last seen.

Desktop notifications say only the kind of request ("Run a command", "Edit a file"), never the command or the path, unless you set `"notifications": {"details": true}` in the config.

### Send and queue

`trailboss send --to <terminal|all> <message>` types one message, then Enter, into agents' terminals (Herdr or tmux). Without `--yes` it only shows who would get it. It types only where the agent is idle or done and still in front; everyone else is skipped with the reason: working, stuck, waiting on its own dialog, no terminal to type into, or `your draft is in the input` (Trailboss reads the input boxes of Claude Code, Codex, Gemini CLI, pi and opencode, and never types over text you left there).

Shortly after typing it checks and says what came of it: `sent` (a turn started), `held` (the agent queued it for later; Trailboss never presses "send now" for you) or `no reaction`. `--no-wait` skips the check. The message itself is never logged or printed.

`--queue` covers agents mid-turn (working, or waiting on you): instead of being skipped, each gets the message once that turn ends, only in the session it was queued for, one message per rest. Queued messages are kept in `send-queue.json` next to the index, readable only by you, and survive a restart. They're typed by a background delivery that `trailboss events`, `watch` or the Herdr block starts, so one of them has to be running; `send --queue` and `trailboss queue` warn when nothing has looked in the last 5 minutes. `trailboss queue` lists what's waiting and the last ones typed, with what came of them; `trailboss queue rm <id>…` (or `--all`) takes them back.

### Until a check passes

`trailboss until <session> -- <check…>` keeps one agent going until a check passes. The session is `AGENT:ID`, a session id or its terminal. After each turn it finishes, Trailboss runs the check in the session's folder, without a shell (`-- sh -c '…'` for a pipeline). On a failure it types the check's last lines and "the check still fails, keep going" into the session; on a pass it stops.

```sh
trailboss until --max-rounds 5 t3 -- cargo test   # Trailboss's flags go before --
trailboss until --max-cost 2 --detach claude:<id> -- npm test
```

- **Limits:** it stops at 10 rounds, $5 spent by the session since the loop started, or 2 hours (`--max-rounds`, `--max-cost`, `--max-hours`); when it stops making progress; or when the session ends.
- **Slow checks:** a check still running after 10 minutes (`--check-timeout`) is killed with everything it started, and counts as a failed round.
- **Limits on your plan:** when a Claude or Codex quota is used up it waits for the reset. A turn that ended on a rate limit isn't a round: it waits for the retry time, then checks once without counting it.
- **Your draft:** it never types over text in a Claude Code, Codex, Gemini CLI, pi or opencode input box; it waits and tries again.
- **Steering:** `trailboss until ls` lists the loops on this machine; `until pause|resume|stop <loop|session|terminal|all>` steers them from any terminal. A paused loop runs nothing and types nothing, and its time doesn't count. `stop` kills a running check. A second loop on the same session is refused.
- **In the background:** `--detach` keeps the loop going after the terminal closes, logging to a file in `until/` next to the index (`until ls` says where). `--json` prints one line per round.

The check is your command and runs as you, in the agent's folder: give it one you'd run there yourself.

### Quotas

`trailboss quota` shows how much of each plan window you've used (Claude's 5-hour and weekly, Codex's), when each resets, and the **pace**: where usage ends up by the reset at the rate so far, flagged ⚠ over 100%. Pace appears once a quarter of the window has passed; before that a single busy session skews it. Over 100%, `limit_in` estimates when you'll hit the limit at this rate. In Herdr, the status block warns once per window when that happens.

This is one of two commands that use the network (the other is the opt-in `trailboss prices --refresh`, under [Cost](#cost)). It calls each provider's own usage endpoint with the sign-in token its CLI already stored (`~/.claude/.credentials.json`, `~/.codex/auth.json`). The token goes only to that provider and is never logged or saved. An expired token is reported, not refreshed: open the agent once and run it again. If the live call fails, Codex falls back to the last limits it logged in a session, marked with their age.

### Delivery stats

`trailboss stats` reads your pull requests from GitHub through `gh`, so `gh` handles sign-in and Trailboss never sees a GitHub token. For the last 30 days (`--days`) it shows merged PRs, median time to merge, open PRs and the oldest one's age. It also shows two measures of AI involvement:

- **`ai_marked`**: PRs whose commits, description or branch name say an AI tool made them (`Co-Authored-By: Claude`, "Generated with Claude Code", a `claude/` branch). PR texts are read only for this and not kept.
- **`with_agent_sessions` and `cost_usd`**: PRs whose branch had agent work, and what it cost. Each session's folder is matched to its GitHub repo through its git remote (worktrees included), and each response to the branch it was made on. `cost_per_pr_usd` is the cost spread over those PRs.

Work done on `main` and moved to a branch only at push time can't be matched to a PR by branch, so the matched cost is a lower bound. The `estimated_*` fields add an estimate of that unmatched work, marked "est." in Herdr, and no work counts for two PRs. Per PR, `estimated_cost_usd` is the exact cost plus its estimated share; the summary's `estimated_cost_usd` and `estimated_cost_per_pr_usd` total and average it. The exact fields stay as they are. `--prs` lists each PR with its cost; the `--json` summary also carries those rows under `prs`, so one call gets both.

Releases (`develop` → `main`) aren't counted, since their work was counted when it merged into `develop`.

Merged PRs from past weeks don't change, so they're kept in `stats-cache.json` next to the index and only the current week is asked for again. The cache holds PR metadata only: repo, number, title, dates, branches and AI marks. Delete the file to fetch everything again. In Herdr, the Stats tab shows the last result straight away, marked with its age, while it asks GitHub.

### Doctor and version

`trailboss doctor` checks what's working and what isn't, one line each with the command that fixes it: the version and the one the installed Herdr mod needs, the index, each source's folder, the Herdr mod's files (against the hashes its installer recorded), layout block and key bindings, tmux, whether `gh` is signed in, whether each provider's quota sign-in is there (by the file's presence only, never reading it), each `checkpoints.auto` folder, and recent failed automatic checkpoints. It only reads, and exits 1 when a check fails.

`trailboss version --json` gives `{version, api}`. `api` goes up only when a `--json` field is renamed or removed, so a frontend can check it works with the CLI it found.

### Sources and config

Trailboss reads Claude Code, Codex (including compressed `.jsonl.zst` rollouts), opencode, pi, Gemini CLI and Copilot CLI with full usage. A Copilot CLI session's tokens are recorded only when it ends. It also lists sessions from about 20 more agents (Cursor, Droid, Kiro, Cline, …) from the start of each session file. `~/.config/trailboss/config.json` (shown and saved, after checking, with `trailboss config [--set '<json>']`) can:

- switch a source off or point it elsewhere: `"sources": {"pi": {"enabled": false}}`;
- add any agent that writes JSONL with token usage, by mapping its fields: `"custom_sources"` (see [custom.rs](https://github.com/trailboss-ai/trailboss-adapters/blob/main/src/sources/custom.rs));
- set the GitHub accounts and work branches `stats` counts: `"github": {"logins": [...], "work_branch": {"owner/repo": "develop"}}`;
- save checkpoints after every turn in some folders (`"checkpoints": {"auto": [...]}`), put task worktrees elsewhere (`"tasks": {"worktrees": "..."}`), or put commands and paths in notifications (`"notifications": {"details": true}`).

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

### Tasks

A task is a brief run by an agent in its own git worktree, merged only when you accept it.

```sh
trailboss task add "Fix the login redirect. Done when: /cart returns to /cart, with a test." --agent codex
trailboss task add --brief-file briefs/ --yes   # one task per .md and .txt file in the folder
```

- **`task add`** shows the plan; with `--yes` it makes a worktree on a new branch `work/<n>-<slug>` off the folder's current branch (`--folder`, default here) and starts the agent in a new Herdr or tmux terminal, nested under the session you ran it from (or `--parent`), with the brief as its first prompt. Your checkout isn't touched. Several briefs (`--brief` or `--brief-file`, repeated) get every worktree made before any agent starts, and a warning when two of them, or a brief and an open task in the same folder, name the same file. A start that fails takes its worktree and branch back.
- **Where worktrees go:** `worktrees/` next to the index by default; `"tasks": {"worktrees": "../worktrees"}` in the config (relative to the repository's top, or absolute), or `--worktrees DIR` for one command. Never inside an agent's own folder.
- **`task ls`:** each task's state (to do, working, needs you, ready, done), branch, commits ahead and files changed.
- **`task show <n>`:** the card to read before accepting: the agent's last message, its commits, each file changed with lines added and removed, checks known to have run (`until` loops on it), and whether `accept` would go through.
- **`task accept <n> --yes`:** fast-forwards the branch the task started from to the task's branch, then removes the worktree and branch. It never makes a merge commit or rebases: it's refused if a fast-forward isn't possible or the checkout has uncommitted changes.
- **`task rm <n> --yes`:** removes the worktree and branch, but keeps commits that aren't merged and uncommitted changes (and says so) unless `--force`.

### Agent definitions

`task add --agent <name>` starts a named definition: a small JSON file, `<name>.json`.

```json
{
  "kind": "codex",
  "description": "Reviews the branch, edits nothing",
  "model": "gpt-5",
  "args": ["--sandbox", "read-only"],
  "rules": "Review only: don't edit files. Report what you'd change and why."
}
```

They're read from a repository's `.trailboss/agents/` (shared with the repo), your `~/.config/trailboss/agents/`, and one built in per agent kind (`claude`, `codex`, `gemini`, `copilot`, `opencode`, `pi`, with no flags); the first with a name wins. `kind` is the agent, `model` its model, `args` its flags and `rules` go before the brief. Flags come only from the allowlist `resume` keeps, each with its values, so a brief can never become a flag. A broken definition (an unknown key, a flag not on the list) is listed with its problem and refused. `trailboss agents` lists them all and which one each name starts; `agents show NAME` shows the command it would run.

A repository's definitions come with the code you cloned and can set flags, including ones that skip an agent's permission prompts. So each must be approved before its first use, and again after it changes: `trailboss agents trust <name> --yes`. Yours and the built-in ones need no approval.

### Checkpoints

A checkpoint is a session's folder saved as a git commit, to go back to.

- **`trailboss checkpoint save AGENT:ID`** (`--label` to name it) commits the folder's tracked files as they are on disk, and untracked ones git doesn't ignore, under `refs/trailboss/checkpoints/`, built in a temporary index, so your index, `HEAD`, branches and stash don't change. Nothing changed since the last one: nothing is saved. Nothing is ever pushed.
- **`checkpoint list AGENT:ID`** lists a session's checkpoints; **`checkpoint diff AGENT:ID N`** shows what N changed.
- **`checkpoint restore AGENT:ID N`** shows which files putting the folder back as N would write and remove. With `--yes` it saves a safety checkpoint first, keeps every file it replaces or removes in `~/.local/share/trailboss/undo/<time>/`, then restores. Ignored files, your index, `HEAD`, branches and stash are left alone. It's refused while the session is working or needs you, or shows no status and isn't known to have finished its turn; `--force` overrides that, with a warning that the agent may still be writing.
- **Automatically:** list folders in the config, `"checkpoints": {"auto": ["~/code/shop"]}`, and a checkpoint is saved whenever a session working there finishes a turn, as `events` or `watch` sees it (so one must be running). Each save runs in the background with a 30-second limit, labelled with the turn's number only, and a session keeps its newest 50. Off by default. Each outcome (saved, unchanged, failed, timed out) is a line in `auto-checkpoints.log` next to the index: the session, the folder and the error, never transcript text.
- **`checkpoint prune [FOLDER] --yes`** drops the checkpoints of sessions that ran in a git worktree since removed (automatic saves do it too).

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
- **Thread** (Alt+Shift+C in an agent's terminal): its conversation beside the terminal, by day, with search, copy, copy code, and quote into the prompt.
- **Install and update:** the mod checks the CLI's version and offers **Install** or **Update** when it's missing or too old, downloading the release to `~/.local/bin` only when you click.
- **Files, editor and Changes** (opt-in): each changed file badged with the agent that changed it; create, rename, move, drag and trash; a multi-tab editor with highlighting, word wrap and find and replace that never overwrites an agent's edit unseen; staging and discarding by hunk; review and undo by session.
- **Spaces with folders** (opt-in): Herdr's spaces list with folders up to four levels deep. Drag spaces in, or let rules file them by folder or machine; fold a folder and it still shows when something in it needs you; show one folder alone.
- It can take the places of the insights, session-history, stats and usage-bar mods in your layouts.

Install, options and details: [trailboss-herdr](https://github.com/trailboss-ai/trailboss-herdr).

## Shape

```
frontends   CLI (JSON out) │ Herdr mod │ your own, over --json
core        trailboss: sessions, usage, fleet, tasks, files
hosts       Herdr, tmux; tuios, experimental
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

- **The `trailboss` binary** is free to use under the [Trailboss License](LICENSE), which also ships with each release. Its source isn't public. Versions 0.1.0 to 0.3.0 were released under the MIT License, and stay so.
- **The adapters, the specification and the Herdr mod** are MIT; see each repository.
- **The name and logo** are trademarks; see [TRADEMARKS.md](TRADEMARKS.md).
