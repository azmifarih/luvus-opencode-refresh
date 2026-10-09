# Luvus OpenCode Refresh

Restart all OpenCode agents in Luvus. Keep every session. Pick up a new OpenCode version.

## Problem

You update OpenCode. The file on disk is new. The agents in Luvus still run the old code. Each pane loads the code into memory when it starts. A new install does not change running panes.

## Solution

This script stops each OpenCode agent in its own pane. It then starts OpenCode again in the same pane. It resumes the same session. The conversation stays complete.

## How It Works

1. The script lists all OpenCode agents with `luvus agent list`.
2. The script always skips its own pane.
3. The script skips blocked panes. Blocked panes need a human.
4. The script skips fresh panes with no session. There is nothing to resume.
5. The script waits until a working pane becomes idle. You can set the wait time. You can also skip working panes.
6. The script sends `/exit` to the pane. The agent stops. The pane returns to shell. The pane ID and the tab stay the same.
7. The script runs `opencode --session <id>` in the same pane. The same session resumes.
8. Some old agents do not know `/exit`. Then the script stops the agent process by session ID. The session data stays safe in the database.

## Requirements

- The `luvus` CLI (https://luvus.dev)
- `jq`
- `pgrep` (only for the fallback in step 8)
- OpenCode V2

## Usage

Show targets without changing anything:

```sh
./luvus-opencode-refresh
```

Wait for idle panes, then restart each pane:

```sh
./luvus-opencode-refresh --apply
```

Restart only idle panes, skip working panes:

```sh
./luvus-opencode-refresh --apply --skip-working
```

### Options

- `--wait-timeout SEC`: maximum wait for one working pane. Default is 1800 (30 minutes).
- `--exit-timeout SEC`: maximum wait for the agent to return to shell. Default is 30.
- `--start-timeout SEC`: maximum wait for the resumed agent to return. Default is 90.
- `--help`: show help.

## Safety Rules

- The script never touches the pane it runs from.
- The script never touches blocked panes.
- If one pane fails, the script leaves it unchanged. It continues with the next pane.
- The script reports a summary at the end: restarted, skipped, failed.

## Verification

I verified this process on a throwaway pane first. I stopped the agent with `/exit`. I resumed the same session in the same pane. The agent remembered the conversation. I then restarted five live panes. All five kept their sessions.
