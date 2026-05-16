# agent-html

A Claude Code slash command that renders the current conversation, plan, or argument as a single self-contained interactive HTML file and opens it in your browser.

Built around the thesis that [HTML is a more honest interchange format than Markdown](https://thariqs.github.io/html-effectiveness/) for ideas you actually want someone to read.

## Usage

```
/html                           # render the current conversation
/html the auth flow             # focus the render on a specific topic
```

The slash command forks the current Claude Code context into a subagent whose only job is to render, save, and open the HTML. The fork inherits the full conversation, so you don't need to repeat anything.

## Install

### Claude Code plugin

```
/plugin marketplace add willness/agent-html
/plugin install agent-html
```

Invocation becomes `/agent-html:html` (plugins get namespaced).

### skills.sh

```
npx skills add willness/agent-html
```

Lands as `/html` in your skills directory.

### Direct clone

```
git clone https://github.com/willness/agent-html.git
ln -s "$PWD/agent-html/skills/html" ~/.claude/skills/html
```

Invocation: `/html`.

### Local dev

```
claude --plugin-dir /path/to/agent-html
```

## Configuration

By default, files are saved to `~/.claude/viz/<YYYY-MM-DD>-<slug>.html`.

Override with the `CLAUDE_HTML_DIR` environment variable. The value supports Mustache-style template tokens.

### Tokens

| Token | Expands to |
|---|---|
| `{{HOME}}` | `$HOME` |
| `{{CWD}}` | shell `pwd` at invocation |
| `{{PROJECT_ROOT}}` | `git rev-parse --show-toplevel` if inside a repo, else `{{CWD}}` |
| `{{REPO_NAME}}` | basename of `PROJECT_ROOT` if inside a repo, else `claude` |
| `{{DATE}}` | today's date, `YYYY-MM-DD` |

### Examples

```bash
# Project-local — files live alongside the repo, gitignorable
export CLAUDE_HTML_DIR='{{PROJECT_ROOT}}/.claude/viz'

# Global stash organized by repo
export CLAUDE_HTML_DIR='{{HOME}}/claude-viz/{{REPO_NAME}}'

# Explicit global default (same as no env var set)
export CLAUDE_HTML_DIR='{{HOME}}/.claude/viz'
```

## What the fork produces

- One self-contained `.html` file. No CDNs, no external assets, no network fetches.
- System fonts. Semantic HTML. A coherent palette.
- SVG diagrams where relationships exist. Interactive widgets where they aid understanding. In-page navigation if the content spans more than one screen.
- Filename is `<YYYY-MM-DD>-<slug>.html`; the slug is picked from the content.

Mobile responsive, dark mode, and print stylesheets are explicitly **not** part of the brief — they cost effort without paying it back here.

## Behavior

- **One-shot.** Each `/html` invocation regenerates a fresh file. To refresh, re-run after editing the conversation.
- **Optional argument** is a focus *hint*, not a filter — the fork still sees the whole conversation.
- **Opens automatically** via `open` (macOS) or `xdg-open` (Linux). Headless or unsupported platform → prints a `file://` URL for manual paste.

## Compatibility

The skill relies on Claude Code's `context: fork` primitive, which forks the parent conversation into a subagent. This is a Claude Code–only feature. The `SKILL.md` is valid and parseable in other skills.sh-compatible agents (Cursor, Codex, Gemini CLI), but they don't have the fork primitive, so the skill won't behave the same way there.

## Credits

- Thariq Shihipar, [*The Unreasonable Effectiveness of HTML*](https://thariqs.github.io/html-effectiveness/)
- Andrej Karpathy, on vision-bandwidth interfaces

## License

MIT. Do whatever you want.
