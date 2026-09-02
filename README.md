# multi-claude

Run several Claude Code accounts side by side on one machine.

The default installation at `~/.claude` is treated as read-only: it is never
modified, moved or relinked. Each extra account lives in `~/.claude-<name>` and
borrows config from the default install through symlinks, so commands, skills,
agents, plugins and `CLAUDE.md` stay in one place.

```sh
multi-claude --claude-work     # create the account and log it in
claude-work                    # launch Claude Code as that account
multi-claude list              # every account and its login state
```

## Install

```sh
install -m 755 multi-claude ~/.local/bin/multi-claude
```

## What is shared, what is not

| Shared with `~/.claude` | Private per account |
|---|---|
| `CLAUDE.md`, `RTK.md`, `settings.json` | `.claude.json` (auth) |
| `commands`, `skills`, `agents`, `plugins` | `history.jsonl`, `sessions` |
| `plans`, `ide`, `agent-memory` | `shell-snapshots`, caches, logs |
| `projects`, `todos`, `session-env` | |

Session state is shared by default so a session started under one account can
be resumed under another. `--no-share-sessions` keeps it private.

## Per-account model

`/model` persists to `settings.json`. While that file is shared, switching the
model in one account switches it in all of them.

`--no-share-settings` gives an account a real `settings.json` instead of the
symlink, so its model stays local:

```sh
multi-claude --claude-work --no-share-settings --no-login
```

`~/.claude/settings.json` remains the single source of truth. Only these keys
stay per account:

    model  effort  fastMode  alwaysThinkingEnabled  outputStyle

Everything else (hooks, permissions, statusline, theme) is rebuilt from the
default install by `sync`:

```sh
multi-claude sync              # every account with a private settings.json
multi-claude sync work         # just one
multi-claude sync --adopt      # fold the account's extra permission rules
                               # back into ~/.claude/settings.json first
```

`--adopt` matters because "always allow" grants land in whichever account you
were running. Without it, a grant made inside `claude-work` is dropped on the
next sync; with it, the rule is promoted to the shared file and reaches every
account.

Sync compares each account against a baseline copy of what it last wrote, not
against the default install. An account that has merely fallen behind is
rewritten silently; only a genuine local edit is reported and backed up before
it is overwritten. `--yes` skips that prompt.

The private choice is sticky: a marker file makes it survive a later
`multi-claude --claude-work` repair run. `--share-settings` reverses it.

## Checking the setup

```sh
multi-claude status work   # what is shared vs private for one account
multi-claude doctor        # verify links, flag accounts needing a sync
```

`doctor` also verifies that nothing inside `~/.claude` points out into an
account directory, and that no account's `.claude.json` has become a symlink,
which would leak one account's credentials into another.

## Removing an account

```sh
multi-claude remove work
```

Signs the account out first, then deletes its config dir and the generated
launcher. Shared targets inside `~/.claude` are never touched, and backups the
script made along the way are left in place for you to delete.
