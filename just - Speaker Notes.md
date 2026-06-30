# `just` — Speaker Notes

**How to read this**

- **Plain bullets are your talking points** — reminders to hit, say them in your own words. Not a script.
- **Lines in a grey box marked `▶ YOU` are instructions to *you*, the presenter** — stage directions, timing, demo cues. **Don't say these out loud.**

---

## 01 · Title

- Ground-up tour of `just`, the command runner
- Starting from zero — not everyone here uses it yet
- By the end: you can write a justfile for any repo, and know the tricks that make it worth adopting

---

## 02 · Agenda

- Six short parts, roughly the hour
- Basics → the big idea (one interface, many repos) → variables & environment → personal vs team → modules → clever recipes

> ▶ **YOU:** 10 minutes saved at the end for Q&A.

---

## 03 · What it is

- A command runner: save your commands as named **recipes** in a `justfile`, run them by name
- NOT a build system — no timestamps, no rebuilds, it just runs commands
- That simplicity is the point

> ▶ **YOU:** Don't assume Make knowledge — reassure anyone who hasn't seen it that they don't need to have.

---

## 04 · The problem

- Every repo invents its own way to do the same handful of tasks
- A new hire clones five repos = five sets of incantations to learn
- Scattered across READMEs, `pom.xml`, Makefiles, CI yaml
- Real cost: day-one friction + tribal knowledge nobody writes down
- `just` turns that into one discoverable file

---

## 05 · Install

- One binary, no runtime, every platform — pick your package manager
- Create a file called `justfile`, run `just` to confirm
- Editor support + shell completions exist (bash / zsh / fish): `just --completions zsh`

---

## 06 · Part 1

> ▶ **YOU:** Section divider — *Part 1: The basics.*

- Recipes, dependencies, parameters — ~5 minutes, but the foundation for everything else

---

## 07 · First recipe

- A recipe = a name, a colon, then indented command lines
- Think of it as a labelled shortcut for commands you'd otherwise type by hand
- Run it with `just <name>`
- That's the whole core idea — everything else is sugar

---

## 08 · Anatomy

- The comment directly above a recipe becomes its docs (shows up in `just --list`)
- Name + optional dependency list come before the colon
- Indented lines are the body — run top to bottom

---

## 09 · List & run

- Discoverability is the killer feature
- Bare `just` (or `--list`) lists every recipe + its doc comment — the repo documents itself
- Common pattern: a `default` recipe that just prints the list
- The `@` prefix silences the command-echo, so output stays clean

---

## 10 · Dependencies

- Dependencies listed after the colon run first, left to right
- Each runs at most once per invocation
- e.g. `just deploy` → install → build & test → the deploy script

> ▶ **YOU:** `&&` runs something *after* the body — mention it, don't dwell.

---

## 11 · Parameters

- Recipes take arguments — declare them after the name, use them with `{{ }}`
- A default makes an argument optional
- `+` = one-or-more, `*` = zero-or-more (forward a list of files or extra flags)
- This is where recipes stop being aliases and become little programs

---

## 12 · Part 2

> ▶ **YOU:** Section divider — *Part 2: One interface, many stacks.*

- The big idea, and the reason most teams adopt `just`
- One consistent interface across repos that are nothing alike underneath

---

## 13 · Same commands

- The payoff slide
- Three repos: a Java service, a Python pipeline, a dbt project — wildly different underneath
- All expose the same three: `just setup` · `just test` · `just deploy`
- The body differs; the interface is identical — the first thing you type is always the same

---

## 14 · Why it matters

- Onboarding collapses to one README line: install just, run just
- No mental reload when context-switching between repos
- CI and local run the exact same recipes → fewer "works on my machine" gaps
- A recipe name is a contract — swap the tool underneath, callers never notice

---

## 15 · Part 3

> ▶ **YOU:** Section divider — *Part 3: Variables & environment.*

- How recipes stop repeating themselves, and how they talk to `.env` and the shell

---

## 16 · Variables

- Assign with `:=` at the top of the file
- They're strings; join them with `+`
- Use them in bodies with `{{ }}` — same syntax as parameters
- e.g. define the version + jar name once, reuse across package and upload

---

## 17 · Functions

- `just` ships a small standard library of functions
- Paths: `justfile_directory`, `invocation_directory`
- Read the environment with a safe default
- `os()` / `arch()` → branch by platform
- Backticks run a shell command at parse time and capture the output (e.g. the current git SHA)

---

## 18 · Dotenv

- Secrets and local config belong in a `.env` file, not the justfile
- `set dotenv-load` → `just` reads `.env` first, each key becomes a shell var (`$DATABASE_URL`)

> ▶ **YOU:** Hammer the gotcha — it's OFF by default. Classic first-time frustration; one line fixes it.

---

## 19 · Exported env

- `just` variables are NOT environment variables by default — interpolation only
- To put one in the environment: `export` a single variable, or `set export := true` for all
- Mental model: `{{NAME}}` = text substitution by just · `$NAME` = a real env var (only if exported)

---

## 20 · CLI overrides

- Variables aren't fixed — callers override them on the command line
- `name=value` before the recipe, or `--set name value`
- Same recipe serves local + CI: default to `dev`, override to `prod` in the pipeline
- No second recipe, no branching

---

## 21 · Part 4

> ▶ **YOU:** Section divider — *Part 4: Personal & shared.*

- Two justfiles serving two different jobs, and keeping them from stepping on each other

---

## 22 · Team justfile

- Lives at the repo root, committed — the shared contract
- setup / test / lint / deploy — written once, owned by everyone, reviewed in PRs
- Doc comments + groups keep `just --list` readable as it grows
- Rule of thumb: would a teammate use it? Then it goes here

---

## 23 · Global justfile

- Your own global justfile — shortcuts that follow YOU between every repo
- `just -g` (`--global-justfile`) looks in your home/config dir, wherever you are
- People alias it to a single letter, `j`
- Personal quality-of-life recipes: a scratch dir, a pretty git log, update-everything
- Never committed to any repo

---

## 24 · Layering

- Bare `just` walks up from your current dir to the nearest justfile — that's the project file
- `just -g` always jumps to the global one
- They never merge — the flag picks the file
- Keep it clean: project recipes are about THIS repo; global recipes are about YOUR workflow
- Copy-pasting a recipe into every repo? It probably belongs in the global file

---

## 25 · Part 5

> ▶ **YOU:** Section divider — *Part 5: Modules.*

- How a justfile stays readable once it has ~forty recipes

---

## 26 · Modules intro

- `mod` pulls in another justfile as a namespace
- Call recipes with the module name first — `just api build`, `just data seed`
- Grouping + namespacing + split ownership, without one 300-line file
- Relatively recent, now a stable feature

---

## 27 · Splitting

- Root justfile declares its modules: `mod api` finds `api.just`
- The module's recipes are namespaced under that name
- `just api build` → build inside api.just · `just data seed` → seed inside data.just
- `just --list` shows the modules and lets you drill in — reads like a directory of capabilities

---

## 28 · Imports

- Sibling concept to modules — people mix the two up
- `import` merges another file's recipes in, no namespace (as if typed inline)
- Perfect for shared boilerplate — a `common.just` with your standard lint/fmt recipes
- `import?` makes it optional — a missing file doesn't error
- Rule of thumb: **modules** organise WITHIN a repo · **imports** SHARE across repos

---

## 29 · Part 6

> ▶ **YOU:** Section divider — *Part 6: Clever recipes.*

- The "oh, I didn't know it could do that" stuff

---

## 30 · Any language

- A recipe body isn't limited to shell — a shebang runs the whole recipe as one script in any interpreter (Python, Ruby, Node…)
- Complex logic stays in a real language instead of being tortured into bash
- Because it's one script, not line-by-line, variables persist across lines

> ▶ **YOU:** This sets up the #1 bash gotcha you'll cover on the Gotchas slide.

---

## 31 · Attributes

- Annotations in `[ ]` above a recipe that change its behaviour
- `[private]` (or a `_` prefix) hides a helper from the list
- `[confirm(...)]` prompts before running — use on anything destructive
- `[group(...)]` tidies `--list` into sections
- `[doc(...)]` sets the description explicitly

> ▶ **YOU:** There are more (no-cd, no-exit-message, working-directory) — these four cover most needs.

---

## 32 · OS-aware

- Define the same recipe name several times, each tagged with an OS attribute
- `just` picks the right one for the current platform — e.g. `open` on Linux / macOS / Windows
- Same name, no if-else inside the body
- Also `[unix]` for the shared case, and a per-OS `set shell`

---

## 33 · Favourites

- `just --show <recipe>` prints a recipe's exact body without running it — "wait, what does this actually do?"
- A self-listing `default` recipe makes the repo self-documenting
- `just --fmt` auto-formats the justfile → clean PR diffs
- `just --dump --dump-format json` emits the whole file as JSON → tools and editors can introspect it

> ▶ **YOU:** Demo one of these live if you have time.

---

## 34 · vs alternatives

- Most teams reach for one of three things today:
  - **The README** → commands copy-pasted out by hand, quietly go stale
  - **Shell aliases** → handy, but live in your personal dotfiles, invisible to teammates
  - **A `scripts/` folder** → better, but no menu and no consistent names
- `just`'s sweet spot: one shared, discoverable set of commands across repos that are nothing alike

---

## 35 · Gotchas

- **1.** Each normal recipe line runs in its OWN shell — a `cd` or variable won't carry to the next line → use a shebang recipe when you need state
- **2.** Dotenv is off by default
- **3.** `{{ }}` is just-interpolation, `$` is the shell — mixing them up is the classic bug
- **4.** Recipes run from the justfile's directory → `[no-cd]` for the caller's directory

> ▶ **YOU:** Land it with "knowing these four saves a lot of head-scratching."

---

## 36 · Cheat sheet

- The whole talk on one page — syntax on the left, the commands you'll actually type on the right

> ▶ **YOU:** Park this slide on screen for Q&A. Tell them to screenshot/pin it.

---

## 37 · Get started

- Don't boil the ocean
- Pick one repo, add three recipes everyone runs daily — setup, test, run
- Reuse the same three names in the next repo
- Add to the README: "install just, then `just`"
- Then let it grow organically — within a month the muscle memory is there and nobody wants to go back

---

## 38 · Q&A

> ▶ **YOU:** 10 minutes for questions. Leave the cheat sheet up, or come back to it.

Likely questions (and your answers):

- **Windows support?** → yes — PowerShell / cmd, via `set shell`
- **CI integration?** → call `just` in your pipeline
- **Secrets?** → `.env` + a secrets manager, never commit
- **Why not Make?** → it's a runner, not a builder

> ▶ **YOU:** Thank everyone.
