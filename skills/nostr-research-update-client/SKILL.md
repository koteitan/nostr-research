---
name: nostr-research-update-client
description: Periodically re-research every Nostr client under client/ and refresh the evidence markdown + README tables. Use when the user asks to update client/ research, refresh client evidence files, or re-run the client investigation. Drives a multi-agent workflow (1 agent per client to research, 1 agent per markdown file to write).
---

# Nostr Client Research Update

This skill refreshes `client/` research: for every client it re-reads the upstream
source, re-derives the four research findings, and rewrites the evidence markdown
files and the two README tables. It is meant to be **run periodically**, so every
step is deterministic and re-runnable.

## What gets produced

```
client/
  README-ja.md          # 5 tables (bootstrap / relay / search / reaction / framework) + index
  README-en.md          # English mirror of README-ja.md
  evidences/
    bootstrap-relay/<client>.md
    relay/<client>.md
    search-relay/<client>.md
    reaction-for-events/<client>.md
```

There are **16 clients × 4 categories = 64 evidence files**, plus the 2 READMEs.
The framework table has **no** evidence files (README-only).

## The four research categories

| Category dir | Question to answer | Typical answer shapes |
|---|---|---|
| `bootstrap-relay` | Which relays does the client connect to *first*, before it knows the user's relay list? (hardcoded defaults, locale-specific additions, env-var defaults) | list of relay URLs + notes |
| `relay` | Where does the client get the relays used to build the **home timeline**? | kind:10002 (NIP-65) / outbox / localStorage settings / cache server |
| `search-relay` | Which relays are used for **full-text search**? | list of relay URLs, or "none" |
| `reaction-for-events` | How are reactions (kind:7, 6, 9735…) for events **collected/crawled**? | subscribe-on-open / batch / per-note / none |

The README also has a **framework** table (language / UI / nostr-access lib / other libs) — no evidence file, filled straight into the README from the research report.

## Client → repo → clone map

All repos are pre-cloned (and `.gitignore`d) under `tmp/clients/<dir>/`. The `<dir>`
name is also the evidence filename stem (`<dir>.md`).

| dir / evidence stem | README display name | upstream repo | source root / notes |
|---|---|---|---|
| nostter | nostter | SnowCait/nostter | `web/` subdir, TS/Svelte |
| rabbit | Rabbit | syusui-s/rabbit | TS/SolidJS |
| lumilumi | Lumilumi | TsukemonoGit/lumilumi | TS/Svelte |
| nos-haiku | Nos Haiku | nikolat/nos-haiku | TS/Svelte |
| nullnull | ぬるぬる | tami1A84/null--nostr | JS/Next.js |
| nosame | 野雨 | invertedtriangle358/Nosame | Vanilla JS |
| flowgazer | flowgazer | ompomz/flowgazer | Vanilla JS |
| yakihonne | Yakihonne | YakiHonne/web-app | JS/React+Next |
| iris | iris | irislib/iris-client | TS/React+Tauri |
| primal | Primal | PrimalHQ/primal-web-app | TS/SolidJS, cache-server based |
| coracle | Coracle | coracle-social/coracle | TS/Svelte, @welshman/* |
| nostrudel | noStrudel | hzrd149/nostrudel | TS/React, applesauce-* |
| amethyst | Amethyst | vitorpamplona/amethyst | Kotlin (Android) |
| damus | Damus | damus-io/damus | Swift+C (iOS), nostrdb |
| algia | algia | mattn/algia | Go (CLI) |
| kakoi | kakoi | betonetojp/kakoi | C# (.NET WinForms) |

The authoritative repo list lives in `client/README-ja.md` → "参考文献" section. If a
client is added/removed there, update this table too.

---

## Phase-2 learnings: how to access the source (READ THIS FIRST)

These are the things that bite you. They were discovered while practicing on nostter.

### 1. Use the pre-cloned repos, but UPDATE them first
The clones in `tmp/clients/<dir>/` are real git clones but **stale** (nostter was 542
commits / 6 months behind on first run). Before researching, update:

```bash
cd tmp/clients/<dir>
DEF=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
DEF=${DEF:-main}                       # fall back to main, then master if needed
git fetch origin
git pull --ff-only origin "$DEF" 2>/dev/null || git pull --ff-only origin master
git log -1 --format='%h %ci %s'        # record this commit hash in the report
```

- `tmp/` is `.gitignore`d, so updating clones never dirties the project repo.
- Use `--ff-only` (non-destructive). Do **not** `git reset --hard` / `checkout` /
  `rebase` — the global git rules forbid those.
- If a clone is missing (new client), create it: `git clone --depth 1 <repo> tmp/clients/<dir>`.

### 2. If you only need one or two files, `gh api` avoids a clone
```bash
gh api repos/<owner>/<repo>/contents/<path> --jq '.content' | base64 -d
```
But for real investigation, the local clone + `grep -rn` is far better — you need to
*discover* where things live, not just read a known file.

### 3. Line numbers DRIFT — never trust the old evidence file's "(行 NNN)"
On nostter the reaction subscription moved from `行 138-139` to line **204** between
runs, and `defaultRelays` lost `relay.nostr.band` while `searchRelays` gained
`nostr.wine`. **Always re-grep the current code** and re-read the snippet; treat the
existing evidence file only as a hint for *where to look*, never as ground truth.
(These drifts are only a reason to re-check the source — they must **not** appear in
the output. See rule 8.)

### 4. Reference URLs use blob links WITHOUT a line anchor
Format: `https://github.com/<owner>/<repo>/blob/<default-branch>/<path-from-repo-root>`.
No `#L123` anchor (anchors rot). Remember path prefixes like nostter's `web/`.

### 5. Where to grep for each category
```bash
cd tmp/clients/<dir>
# bootstrap / relay / search relays — usually one constants/config file:
grep -rn "defaultRelay\|bootstrapRelay\|localizedRelay\|searchRelay\|metadataRelay" src
# home-timeline relay source — NIP-65 / outbox / localStorage:
grep -rn "10002\|RelayList\|outbox\|kind.*3\b\|localStorage" src
# reactions — kind 7 and friends:
grep -rn "kinds:.*7\|Reaction\|9735\|'#e'" src
```
Adjust `src` to the client's source root (`web/src` for nostter, repo-specific for
native clients). For Damus/Amethyst/algia/kakoi the language differs (Swift/Kotlin/Go/C#)
— grep for the same concepts (kind numbers, relay-list, NIP-65) in their idioms.

### 6. Some answers are legitimately "none" / "cache-server"
e.g. 野雨 & algia do not fetch reactions ("送信のみ"); Primal routes everything through
`cache.primal.net`; Damus has no full-text search. Record these as the conclusion, not
as a failure.

### 7. Dates
Use today's date (see project `CLAUDE.md` → currentDate) for the README
`*最終更新: YYYY-MM-DD*` lines and any evidence "調査日".

### 8. Keep diff-from-previous-research OUT of the summary READMEs (evidence files MAY keep it)
The two summary pages — `client/README-ja.md` and `client/README-en.md` — must describe
how each client works **now**, not what changed since the last run. Every table cell
must read as a standalone description of the current implementation. **Forbidden in the
READMEs**: any wording that references a past version or the previous research, e.g.
"前回調査から変更", "旧版にあった〜は削除", "〜が追加された", "removed since the previous
research", "no longer", "now uses".
- Bad (README):  `relay-jp…, yabu.me, r.kojira.io（前回調査の relay.barine.co が削除され nostr.compile-error.net が追加された）`
- Good (README): `relay-jp…, yabu.me, r.kojira.io, nostr.compile-error.net`
- Bad (README):  `nos.lol が削除され relay.primal.net が追加された`
- Good (README): `temp.iris.to, vault.iris.to, relay.damus.io, relay.snort.social, relay.primal.net`

The **evidence files** (`client/evidences/**`) are detailed investigation pages and MAY
include a short diff note (e.g. a "前回調査から…" bullet in the 説明 section) when a change
is worth recording. Carry that diff text in the report's `changeNote` field (below) so
the evidence writer can use it while the README writer drops it. So: diff notes are
allowed in evidence files, never in the two README summary pages.

---

## Phase 3 — research: **1 sub-agent per client**

Spawn one agent per client (16 total). Each agent:

1. Reads this SKILL.md.
2. Updates its client's clone (Phase-2 step 1) and records the HEAD commit hash.
3. For each of the 4 categories, greps + reads the current source and determines the
   answer, capturing: **conclusion**, **source file path** (from repo root),
   **line range** (from *current* code), a short **code snippet**, the **reference
   blob URL**, **notes**, and an optional **changeNote**. `conclusion` and `notes`
   feed the README, so they describe the **current** state only — no diff wording
   (see Phase-2 rule 8). Put any diff-from-previous-research observation in
   `changeNote` (evidence-only; may be empty).
4. Also captures the **framework** row: language, UI, nostr-access lib, other libs.
5. Returns a single structured report (use the schema below).

The agent must **not** write any files in Phase 3 — it only researches and returns data.
This separation keeps research (code-reading) and writing (formatting) independent and
lets the README tables be assembled only after every client is done.

### Phase-3 report schema (per client)

```json
{
  "dir": "nostter",
  "displayName": "nostter",
  "repo": "https://github.com/SnowCait/nostter",
  "defaultBranch": "main",
  "headCommit": "1f8cf9d",
  "categories": {
    "bootstrap-relay":     { "conclusion": "...", "file": "web/src/lib/Constants.ts", "lines": "95-124", "snippet": "...", "refUrl": "https://github.com/SnowCait/nostter/blob/main/web/src/lib/Constants.ts", "notes": "日本語設定時に日本リレー追加 …", "changeNote": "前回調査時の relay.nostr.band がデフォルトから外れた（evidence-only, may be empty）" },
    "relay":               { "conclusion": "kind:10002 (NIP-65)", "file": "...", "lines": "...", "snippet": "...", "refUrl": "...", "notes": "", "changeNote": "" },
    "search-relay":        { "conclusion": "...", "file": "...", "lines": "...", "snippet": "...", "refUrl": "...", "notes": "", "changeNote": "" },
    "reaction-for-events": { "conclusion": "...", "file": "...", "lines": "...", "snippet": "...", "refUrl": "...", "notes": "", "changeNote": "" }
  },
  "framework": { "language": "TypeScript", "ui": "Svelte + SvelteKit", "nostrAccess": "rx-nostr, nostr-tools, @rust-nostr/nostr-sdk", "otherLibs": "rxjs, Melt UI, svelte-i18n" }
}
```

---

## Phase 4 — write: **1 sub-agent per markdown file**

After all Phase-3 reports are in, spawn writer agents. **One agent writes exactly one
markdown file** (this is the "1 sub-agent 1 md" rule).

- **64 evidence writers**: each gets one `(client, category)` finding from the reports
  and writes `client/evidences/<category>/<dir>.md` using the template below.
- **2 README writers**: one for `client/README-ja.md`, one for `client/README-en.md`.
  Each receives **all** reports and rebuilds the 5 tables + the index/参考文献 sections.

A writer overwrites its one file with `Write` and does nothing else, so writers never
conflict (each owns a distinct path). README-ja and README-en are separate files →
their two writers are independent.

### Evidence file template (Japanese — `*.md`)

```markdown
← [README](../README.md)

# <displayName> <category-title-ja>

## 結論
- **<category-label>**: <conclusion>
<optional extra bullet lines, e.g. 日本語設定時追加: …>

## ソースコード

**ファイル**: `<file>` (行 <lines>)

```<lang>
<snippet>
```

## 説明
- <explanatory bullets — keep concise, derived from the code, describing the current behaviour>
- <if changeNote is non-empty, add ONE diff bullet here, e.g. "前回調査から…"; evidence files
  may carry this, the READMEs may not (see rule 8)>

## 参考
- <refUrl>

---
← [README](../README.md)
```

`<category-title-ja>` / `<category-label>` per category:
- bootstrap-relay → "Bootstrap Relay" / "Bootstrap リレー"
- relay → "リレー取得方法" / "リレー取得方法"
- search-relay → "検索リレー" / "検索リレー"
- reaction-for-events → "リアクション取得方法" / "リアクション取得方法"

English evidence files were historically not maintained per-category (only the READMEs
are bilingual). Keep evidence files Japanese unless an existing `*-en.md` is present.

### README tables (both languages)

Rebuild these five tables from the reports, preserving the existing column layout
(see current `client/README-ja.md` for the exact headers and the index/参考文献 blocks).
Build each cell from `conclusion` + `notes` only — **never** include `changeNote`, and
strip any stray diff-from-previous-research wording so every cell is current-state only
(see rule 8):

1. Bootstrap リレー — `bootstrap-relay` conclusion + notes, links to evidence file.
2. リレー — `relay` conclusion + notes.
3. 検索リレー — `search-relay` conclusion.
4. リアクション — `reaction-for-events` conclusion.
5. フレームワーク — framework row (language / UI / nostr-access / other libs), **no** link.

Update every `*最終更新: YYYY-MM-DD*` line to today's date. Keep the top back/lang link
line (`[<< back](../README-ja.md) | [Japanese] | [English](README-en.md)`) and the
"調査済みクライアント一覧" + "参考文献" sections intact (refresh repo URLs only if changed).

---

## How to run it (the workflow)

Run via the **Workflow** tool. Skeleton:

```js
export const meta = {
  name: 'update-client-research',
  description: 'Re-research every Nostr client and rewrite evidence + README tables',
  phases: [
    { title: 'research', detail: '1 agent per client' },
    { title: 'write-evidence', detail: '1 agent per evidence md' },
    { title: 'write-readme', detail: '1 agent per README' },
  ],
}

const SKILL = 'skills/nostr-research-update-client/SKILL.md'
const CLIENTS = [ /* the 16 dirs from the map above */ ]
const CATEGORIES = ['bootstrap-relay','relay','search-relay','reaction-for-events']

phase('research')
const reports = (await parallel(CLIENTS.map(c => () =>
  agent(`Read ${SKILL}. Follow "Phase 3" for client dir "${c}". Update its clone, research all 4 categories + framework, return the report.`,
        { label: c, phase: 'research', schema: REPORT_SCHEMA })
))).filter(Boolean)

phase('write-evidence')
await parallel(reports.flatMap(r => CATEGORIES.map(cat => () =>
  agent(`Read ${SKILL} "Phase 4". Write client/evidences/${cat}/${r.dir}.md from this finding: ${JSON.stringify(r.categories[cat])} (client "${r.displayName}").`,
        { label: `${cat}/${r.dir}`, phase: 'write-evidence' })
)))

phase('write-readme')
await parallel([
  () => agent(`Read ${SKILL} "Phase 4". Rebuild client/README-ja.md 5 tables from: ${JSON.stringify(reports)}`, { label: 'README-ja', phase: 'write-readme' }),
  () => agent(`Read ${SKILL} "Phase 4". Rebuild client/README-en.md 5 tables (English) from: ${JSON.stringify(reports)}`, { label: 'README-en', phase: 'write-readme' }),
])
```

Research is a barrier (README needs all clients). Evidence + README writers run after.
No worktree isolation is needed: every research agent touches a different clone dir and
every writer owns a different output file.

## After the workflow
- `git diff --stat client/` to review scope.
- Spot-check 2-3 evidence files against the live source for line-number/snippet accuracy.
- Commit per the project commit rules (Japanese message; do not `git add -A`).
