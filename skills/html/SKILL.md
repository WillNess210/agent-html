---
name: html
description: Render the current conversation, plan, idea, or argument as a single self-contained interactive HTML file and open it in the user's browser. Invoke with /html (optionally /html <focus hint>). Auto-invoke when the user asks to visualize as HTML, "make an HTML page from this", "render this as a webpage", "show this as an interactive page", or wants a richer visual artifact than markdown can express.
argument-hint: [optional focus hint, e.g. "the auth flow"]
allowed-tools: Bash, Write, Read
---

# /html — render the conversation as interactive HTML

Your single job in this turn: produce one self-contained HTML file that captures the current plan / idea / argument from the conversation as a rich visual artifact, save it to disk, open it in the user's browser, and report the path. Don't ask follow-ups — go.

## Why HTML, not Markdown

A document the user *skims* is worse than one they *read*. HTML gives you SVG diagrams, structured layout, visual hierarchy, interactive widgets, and in-page navigation — none of which Markdown can express. Modern context windows removed the token-scarcity argument that once justified plain text. Spend the tokens. Use the user's vision bandwidth — roughly a third of human cortex — instead of their patience.

Reference points:
- Thariq Shihipar, *The Unreasonable Effectiveness of HTML*
- Examples: https://thariqs.github.io/html-effectiveness/
- Karpathy on vision-first interfaces

## Argument

`$ARGUMENTS`, if non-empty, is a **focus hint** — what the user wants emphasized. It does *not* filter the conversation; render against the full context. If empty, infer the most important artifact in the conversation and render that.

## Output: hard constraints

- **Exactly one `.html` file.** No external CSS, no external JS, no CDN links, no `<img src="http…">`, no Google Fonts, no network fetches at runtime. Everything embedded.
- **System fonts only.** Use `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif` (and the monospace equivalent for code).
- **Semantic HTML.** Real `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>` where they fit.
- **A `<title>` reflecting the content.** Not "Document" or "Plan".
- **Clear visual hierarchy.** Headings size down meaningfully; whitespace is generous; line length sits around 65ch for body text.
- **In-page navigation** if the content spans more than ~one screen — sticky TOC, anchor links, or both.

## Output: strongly preferred

- **SVG diagrams** wherever relationships, flows, hierarchies, sequences, or comparisons exist. Hand-author the SVG inline; never embed raster images.
- **Interactive widgets** where they aid understanding: collapsible details, tabs, toggles, click-to-expand sections, hover/focus reveals. Use plain `<details>` for the simple cases; small vanilla JS for the rest.
- **A coherent palette.** Neutral background, one accent color. Avoid the generic "AI-generated webpage" look.

(Mobile responsive, dark mode, and print stylesheets are explicitly **not** required.)

## Save location

Resolve the output directory in this order:

1. If `$CLAUDE_HTML_DIR` is set, use it after expanding Mustache-style tokens.
2. Otherwise, default to `~/.claude/viz/`.

**Supported tokens** (expand in the env-var value):

| Token | Expands to |
|---|---|
| `{{HOME}}` | `$HOME` |
| `{{CWD}}` | output of `pwd` |
| `{{PROJECT_ROOT}}` | `git rev-parse --show-toplevel 2>/dev/null` if inside a repo, else `{{CWD}}` |
| `{{REPO_NAME}}` | basename of `PROJECT_ROOT` if inside a repo, else `claude` |
| `{{DATE}}` | today's date, `YYYY-MM-DD` |

**Filename:** `<YYYY-MM-DD>-<slug>.html`. Pick a short kebab-case slug from the content (e.g. `auth-flow`, `q3-roadmap`, `html-skill-design`). Avoid generic slugs like `plan` or `document`.

**Concrete steps:**

```bash
# 1. Resolve directory
raw="${CLAUDE_HTML_DIR:-$HOME/.claude/viz}"
home="$HOME"
cwd="$(pwd)"
project_root="$(git rev-parse --show-toplevel 2>/dev/null || echo "$cwd")"
repo_name="$(basename "$project_root" 2>/dev/null || echo claude)"
date="$(date +%Y-%m-%d)"

dir="${raw//\{\{HOME\}\}/$home}"
dir="${dir//\{\{CWD\}\}/$cwd}"
dir="${dir//\{\{PROJECT_ROOT\}\}/$project_root}"
dir="${dir//\{\{REPO_NAME\}\}/$repo_name}"
dir="${dir//\{\{DATE\}\}/$date}"

mkdir -p "$dir"

# 2. Build the absolute output path (you choose <slug>)
path="$dir/$date-<slug>.html"
```

Write the HTML to `$path` using the Write tool with the absolute path.

## Open in browser

After the file is written, run this helper. macOS uses `open`; Linux uses `xdg-open`; everything else falls back to printing a `file://` URL.

```bash
open_file() {
  local file="$1"
  [[ "$file" != /* ]] && file="$(cd "$(dirname "$file")" && pwd)/$(basename "$file")"
  if [ -z "$DISPLAY" ] && [ -z "$WAYLAND_DISPLAY" ] && [ "$(uname -s)" != "Darwin" ]; then
    printf "file://%s\n" "$file"; return 0
  fi
  case "$(uname -s)" in
    Darwin) open "$file" ;;
    Linux)  xdg-open "$file" 2>/dev/null || printf "file://%s\n" "$file" ;;
    *)      printf "file://%s\n" "$file" ;;
  esac
}
open_file "$path"
```

## Report back

Return exactly one line:

```
Rendered → <absolute path>
```

Nothing else. No summary of what's on the page. The user is already looking at it in the browser.

## One-shot

There is no iteration loop. Render once, open, report. If the user wants changes, they re-run `/html`. Don't offer to "make adjustments" inside this invocation — return control to the user immediately.
