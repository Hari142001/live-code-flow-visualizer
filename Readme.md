# Live Code Flow Visualizer

An interactive, single-file web app that shows **how Python code runs, step by step** — with a generated flowchart, live variable tables, call stack, and side-by-side real execution. Built for learning, teaching, and debugging beginner code.

> **Live demo:** `https://<your-username>.github.io/live-code-flow-visualizer/`
> (replace `<your-username>` after enabling GitHub Pages)

---

## What it does

Paste or type Python code, hit **Run**, and the app:

1. **Parses** the code and draws a **flowchart** of its structure (if/else, loops, functions, calls, returns).
2. **Traces** execution step by step — the highlight moves through the flowchart and the source lines as the program runs.
3. **Shows live state**: variables and their types, the current call stack, and each frame's locals.
4. **Actually runs the code** in parallel so you see the real `stdout`, real `stderr`, and real exceptions — not just a simulation.

The tracer is intentionally simple and pedagogical. It understands the subset of Python that beginners and coding-interview learners use most. Anything it doesn't model still **executes for real**, so the output is never wrong — the step-by-step view just falls back to a generic "opaque" step.

---

## Features

- **Step controls** — Restart, Prev, Play/Pause, Next, plus **Step Into / Step Over / Step Out** for function calls.
- **Adjustable playback speed** with a slider.
- **Flowchart** with colored nodes: executed (green), current (yellow), calls (purple dashed), returns (orange), unmodeled constructs (dashed outline).
- **Variables tab** — global + current local scope, with changed values highlighted.
- **Call Stack tab** — top-of-stack first, plus a running list of calls and returns.
- **Execution Flow tab** — a readable trace log of every step.
- **Preview tab** — sandboxed live render for HTML and CSS.
- **Input tab** — one value per line for `input()`.
- **Output / Errors / Logs / Help** — real stdout, real error text, trace log, and an error assistant.
- **Error assistant** — plain-English explanations for common Python mistakes (`IndentationError`, `EOFError`, `ZeroDivisionError`, `IndexError`, `NameError`, missing `:`, etc.) with a **previewed, opt-in fix** you can apply with one click.
- **Editor niceties** — line numbers, current-line highlight, bracket auto-close, auto-indent, Tab inserts 4 spaces, **autocomplete** for keywords, builtins, snippets, and identifiers already in your file.
- **Format** button — re-indents Python/HTML, braces for braces-based languages.
- **Theme toggle**, **Download**, **Reset**, and automatic **autosave** per language via `localStorage`.

---

## Supported languages

| Language | Step trace & flowchart | Real execution |
| --- | --- | --- |
| **Python** | ✅ Full | ✅ Pyodide (WASM) — runs in your browser |
| **HTML** | — | ✅ Rendered preview |
| **CSS** | — | ✅ Applied to demo page |

**Python is the star of the show**: the flowchart, step trace, variables, and call stack are Python-only. HTML and CSS are supported for editing and previewing inside the sandbox.

---

## Getting started

The app is a single `index.html` with no build step and no dependencies to install.

```text
Double-click index.html
```

That's it. **Python works out of the box** — the first Run downloads the Pyodide runtime (~10 MB) from a CDN. Subsequent runs reuse it.

> For the best experience use a modern browser (Chrome, Edge, Firefox, Safari). Web Workers and WebAssembly must be enabled — they are by default.

---

## Deploying live (free)

Because the app is a static single file, **GitHub Pages is the easiest free host**.

1. Push this repo to GitHub (branch `main`, `index.html` at the root).
2. Repo → **Settings → Pages**.
3. **Source:** Deploy from a branch
   **Branch:** `main`   **Folder:** `/ (root)`
4. **Save**. After ~1 minute your site is at:

   ```text
   https://<your-username>.github.io/live-code-flow-visualizer/
   ```

Everything works on GitHub Pages:
- **Python** ✅ (Pyodide loads from a public CDN)
- **HTML / CSS** ✅ (preview is sandboxed)

Alternative free static hosts that also work with zero config: Cloudflare Pages, Netlify, Vercel.

---

## How it works (architecture)

Everything lives in `index.html`:

- **`<script id="eng">`** — the mini-engine:
  - A **tokenizer** and **Pratt-style expression parser** for the Python subset the tracer models.
  - A **statement parser** for `if/elif/else`, `for`, `while`, `def`, `return`, `break`, `continue`, `pass`, `import`, assignment, augmented assignment, `print(...)`, calls, and one-line blocks.
  - A **flowchart builder** that lays out nodes and edges from the AST.
  - A **tracer** that executes the AST and records a step for every meaningful action (condition result, loop iteration, call, return, variable change, I/O).
- **`pyWorkerMain()`** — a Web Worker that loads **Pyodide** from a CDN and runs the real Python code, capturing `stdout`, `stderr`, the file written to `os.environ['OUTPUT_PATH']`, and any exception traceback. It auto-loads Pyodide packages from imports and tries `micropip` for pure-Python PyPI packages.
- **Main UI** — the editor, tabs, controls, autocomplete, error assistant, and rendering.

Because the tracer and Pyodide run in **separate Web Workers**, execution can't freeze the UI, and the 8-second tracer timeout / 20–120-second sandbox timeout protect against runaway loops.

**Design choice:** the visualizer never claims to be Python. It's a *model* of Python aimed at learners. Real execution is the source of truth; the trace is the explanation layer on top.

---

## Files

```text
live-code-flow-visualizer/
├── index.html      # the whole app (UI + engine + Pyodide worker)
└── README.md
```

---

## Keyboard shortcuts

| Shortcut | Action |
| --- | --- |
| `Tab` | Insert 4 spaces |
| `Enter` | Auto-indent (and auto-close `:` blocks) |
| `(`, `[`, `{`, `"`, `'` | Auto-close pair |
| `Ctrl` + `Space` | Force-show autocomplete |
| `↑` / `↓` | Move in autocomplete list |
| `Enter` / `Tab` | Accept autocomplete |
| `Esc` | Dismiss autocomplete |
| `Ctrl` + `Z` | Undo (including an applied error fix) |

---

## Limitations

- **The tracer models a subset of Python.** `class`, `try/except`, `with`, comprehensions, generators, decorators, `match`, and multi-assignment (`a, b = ...`) are not step-traced. They still run for real, and the trace shows a single "opaque" step for them.
- **Flowchart layout is simple** — branches go left/right, loops go down and back. Very deeply nested code can get wide.
- **`print` / `input` are recognized as builtins**, not first-class AST nodes. Aliased or wrapped calls still run but aren't highlighted specially.
- **Pyodide is loaded from a CDN** (`cdn.jsdelivr.net`). Offline use requires vendoring it.
- **No persistence across browsers.** Autosave is per-browser `localStorage`.

---

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| Python first run hangs for a while | Pyodide is downloading (~10 MB). Wait once; later runs are instant. |
| *"EOFError: EOF when reading a line"* | Your program called `input()` but the **Input** tab is empty. Add one value per line and Run again. |
| Flowchart doesn't appear | The code has a parse error the visualizer can't model. Check the **Errors** tab — real Python errors are shown there. |
| `git init` fails inside `Documents` on Windows | OneDrive is locking the folder. Keep code in `C:\dev` (or any non-synced path). |

---

## Roadmap ideas

- [ ] Multi-assignment tracing (`a, b = f()`)
- [ ] `try/except` as a first-class node
- [ ] Better loop-body layout for long blocks
- [ ] Export flowchart as SVG / PNG
- [ ] Shareable permalinks via URL fragment

---

## License

MIT — do what you like, attribution appreciated.

---

## Credits

- **Pyodide** for in-browser CPython: <https://pyodide.org>
- Inspired by Python Tutor, but built around a self-contained static single file.