---
name: nostr-research-update-relay
description: Periodically re-research the Nostr relay implementations under relay/ and the live relay instances under relay-instances/, then refresh their evidence markdown + README tables. Use when the user asks to update relay/ research, refresh relay evidence, re-query relay-instances NIP-11 info, or re-run the relay investigation. Drives a multi-agent workflow (1 agent per relay/instance to research, 1 agent per markdown file to write).
---

# Nostr Relay Research Update

This skill refreshes **two independent areas**. It mirrors the
`nostr-research-update-client` skill (Phase 3 research → Phase 4 write, 1 agent per
unit, 1 agent per markdown file) but adapts to relay-specific facts. Both areas are
meant to be **run periodically**, so every step is deterministic and re-runnable.

- **Part A — `relay/`**: source-code research of 6 relay *implementations*. Findings are
  **version-pinned to a specific commit**. Produces one big multi-section README (ja+en)
  and bilingual evidence files.
- **Part B — `relay-instances/`**: **live NIP-11 queries** against ~11 running relay
  *instances*. No source, no clones — just an HTTP request per instance. Produces one
  NIP-11 limitation table (ja+en).

Run them as two separate workflows; they share nothing.

## What gets produced

```
relay/
  README-ja.md            # one document: 目次 + ranking + limit/rate/filter+size/time tables + 参考文献 + version info
  README-en.md            # English mirror
  evidences/
    <relay>.md            # Japanese, per relay
    <relay>-en.md         # English, per relay   (BILINGUAL — 6 relays × 2 = 12 files)
relay-instances/
  README-ja.md            # one NIP-11 limitation table
  README-en.md            # English mirror
                          # (no evidences/, no clones)
```

---

# Part A — `relay/` implementation research

## Relay → repo → clone map (6)

All repos are pre-cloned (and `.gitignore`d) under `tmp/relay/<dir>/`. The `<dir>` name
is the evidence filename stem (`<dir>.md` + `<dir>-en.md`).

| dir / evidence stem | repo | lang | where to look (config / source) | base |
|---|---|---|---|---|
| strfry | hoytech/strfry | C++ | `strfry.conf` | - |
| nostream | cameri/nostream | TypeScript | `resources/default-settings.yaml` | - |
| nostr-rs-relay | scsibug/nostr-rs-relay | Rust | `config.toml`; limit behaviour in `src/repo/sqlite.rs` | - |
| khatru | fiatjaf/khatru | Go (framework) | `relay.go`, `policies/sane_defaults.go`, `responding.go` (no config file) | - |
| haven | bitvora/haven | Go | `.env.example`, `limits.go`, `init.go` | khatru |
| wot-relay | bitvora/wot-relay | Go | `.env.example`, `main.go` | khatru |

The authoritative repo list lives in `relay/README-ja.md` → "参考文献". The pinned commit
of each relay lives in `relay/README-ja.md` → "バージョン情報".

## Research dimensions (per relay)

These map 1:1 to the sections of an evidence file (`relay/evidences/strfry.md` is the
reference template). Capture each with a **version-pinned reference URL**:

1. **概要** — language, config file, repo, confirmed commit + date.
2. **limit パラメータ** — default max `limit`, config param, behaviour, config snippet.
3. **レート制限** — max subscriptions, event-send rate, filter/REQ rate, connection rate.
   (nostream has one row per kind-group; haven has one row per relay type — irregular.)
4. **時間ベースの制限** — `created_at` validation (max future / past offset) and
   storage/deletion policy (ephemeral age, ephemeral lifetime, normal max age).
5. **フィルター値制限** — filter-value limit, max filters per REQ, approx max authors.
6. **サイズ制限** — event size, WebSocket payload, tag-value size, max tags, content size.
7. **サポートNIP** — the NIP list.

## Phase-2 learnings (relay-specific — READ THIS FIRST)

### 1. Update the clone first; default branch VARIES
Clones in `tmp/relay/<dir>/` are stale (strfry was 455 commits behind on first run).
**strfry's default branch is `master`**, not main — detect it, don't assume:

```bash
cd tmp/relay/<dir>
DEF=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
DEF=${DEF:-main}
git fetch origin
git pull --ff-only origin "$DEF" 2>/dev/null || git pull --ff-only origin master
git log -1 --format='%h %H %ci'    # record SHORT hash, FULL hash, and date
```
`--ff-only` only (never `reset --hard`/`checkout`/`rebase` — global git rules). If a
clone is missing: `git clone <repo> tmp/relay/<dir>`.

### 2. VERSION-PINNING is the whole game (the big difference from client research)
Unlike client research (which links to `blob/<branch>/…`), every relay reference URL is
pinned to a **short commit hash**, e.g.
`https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91`, **with a `#Lnn`
anchor**. So on every update you must:
- record the new short+full commit hash after the pull,
- **re-grep every referenced parameter** to get its CURRENT line number (line numbers
  drift — strfry's params sit at L76/79/91/94 today but will move), and
- rewrite ALL blob URLs to the new hash + new line, AND update the per-relay commit
  hashes in `relay/README-ja.md` / `README-en.md` → "バージョン情報" and each evidence
  file's "確認バージョン".

A reference URL whose hash and the doc's pinned hash disagree is a bug.

### 3. nostr-rs-relay URL quirk + SSH remote
Existing nostr-rs-relay links use an unusual sourcehut-style path
`…/scsibug/nostr-rs-relay/tree/<hash>/item/config.toml#L138`. Normalize to the github
blob form `https://github.com/scsibug/nostr-rs-relay/blob/<hash>/config.toml#L138`. Its
clone's `origin` is an **SSH** URL (`git@github.com:…`); if `git fetch` fails on auth,
re-point with `git remote set-url origin https://github.com/scsibug/nostr-rs-relay.git`.

### 4. khatru is a framework; haven & wot-relay inherit from it
For values haven/wot-relay don't override, the answer is "khatru継承" / "Inherited from
khatru" (e.g. MaxMessageSize 500 KB). khatru itself has no config file — facts come from
Go source (`relay.go`, `policies/sane_defaults.go`). State framework-level defaults via
`ApplySaneDefaults`.

### 5. "制限なし" / "未設定" / "該当なし" are valid answers
e.g. nostr-rs-relay has no default max limit; strfry sets no default event-send rate.
Record the absence, not a failure.

### 6. The README has two LIVE-DATA bits beyond the source
- **リレーランキング** table comes from <https://nostr.watch/relays/software>. Its date is
  a **Last Checked** date, not a last-changed date — it expresses how recently the ranking
  was verified, so **always set it to the run date (today)**, even when the numbers are
  unchanged. Label it `(Last Checked: YYYY/MM/DD)` in both READMEs. Try to refetch the
  numbers (nostr.watch is a JS SPA, hard to scrape) and refresh them if you can; if not,
  carry the prior table forward but still stamp today's Last Checked date.
- **バージョン情報** — bump `ドキュメントバージョン`, set the `Last Checked` line to today,
  and list each relay's full commit hash + date from step 1.

### 7. Dates — use "Last Checked" semantics
Every survey/verification date in these docs is a **Last Checked** date (when the research
was last run), not a last-changed date — always stamp it with today's date (project
`CLAUDE.md` → currentDate) even when nothing changed, to express how fresh the check is.
This covers the バージョン情報 `Last Checked` line, the `リレーランキング (Last Checked: …)`
heading, and (Part B) the relay-instances `Last Checked:` line. The evidence files'
"確認バージョン" is a different thing — that carries the pinned commit + its commit date.

### 8. Keep diff-from-previous-research OUT of the summary READMEs (evidence files MAY keep it)
Same rule as the client skill: `relay/README-ja.md` and `relay/README-en.md` describe the
current pinned version only — no "前回から変更" / "removed since" wording in any cell or
prose. The **evidence files** MAY carry a short diff note. Carry diff text in the report's
`changeNote` (evidence-only); the README writer drops it.

## Phase 3 — research: **1 sub-agent per relay** (6 agents)

Each agent: reads this SKILL, updates its clone (learning 1), records the commit, then
greps the config/source for all 7 dimensions, capturing version-pinned `refUrl`s
(learning 2). It writes **no files** — it only returns the report below. Keep
`conclusion`/scalar facts current-state only; put any diff in `changeNote`.

### Phase-3 report schema (per relay)

```json
{
  "dir": "strfry", "displayName": "strfry", "repo": "https://github.com/hoytech/strfry",
  "base": "", "language": "C++", "configFile": "strfry.conf",
  "defaultBranch": "master",
  "headCommitShort": "542552ab", "headCommitFull": "542552ab…", "headDate": "2025-01-10",
  "limit":        { "defaultMax": "500", "configParam": "relay.maxFilterLimit", "behavior": "…", "snippet": "…", "refUrl": "https://github.com/hoytech/strfry/blob/542552ab/strfry.conf#L91" },
  "rateLimit":    { "rows": [ { "type": "-", "maxSubs": "20", "eventRate": "未設定", "filterRate": "未設定", "connRate": "未設定", "notes": "全て設定可能", "refUrls": { "maxSubs": "…#L94" } } ] },
  "time":         { "futureOffset": "+900秒 (15分)", "pastOffset": "-94,608,000秒 (約3年)", "ephemeralAge": "60秒", "ephemeralLifetime": "300秒", "normalMaxAge": "-", "notes": "Kind 20000-29999のみ", "refUrls": { } },
  "filterValues": { "filterValueLimit": "制限なし", "maxFiltersPerReq": "200", "approxMaxAuthors": "~1,900 (WebSocket制限による)", "configParam": "relay.maxReqFilterSize", "notes": "…", "refUrls": { } },
  "size":         { "eventSize": "65,536 (64 KB)", "wsPayload": "131,072 (128 KB)", "tagValueSize": "1,024", "maxTags": "2,000", "contentSize": "-", "configParams": "events.maxEventSize, relay.maxWebsocketPayloadSize", "refUrls": { } },
  "supportedNips": "1, 2, 4, 9, 11, 22, 28, 40, 70, 77",
  "changeNote": ""
}
```

Use `""` / `"-"` / `"制限なし"` where a field does not apply. Irregular rate-limit shapes
(nostream per-kind, haven per-type) go in `rateLimit.rows[]` as multiple rows.

## Phase 4 — write: **1 sub-agent per markdown file**

After all 6 reports are in, spawn writers. **One agent writes exactly one md file.**

- **12 evidence writers** — `<relay>.md` (Japanese) and `<relay>-en.md` (English) for each
  relay. Each gets one relay's report and reproduces the existing evidence layout
  (`relay/evidences/strfry.md` / `strfry-en.md` are the exact templates: 概要 / limit /
  レート制限 / 時間ベース / フィルター値 / サイズ / サポートNIP, with the
  `[<< back] | [Japanese] | [English]` header+footer and version-pinned links). The
  `*-en.md` is a full English mirror.
- **2 README writers** — `relay/README-ja.md` and `relay/README-en.md`. Each receives
  **all 6 reports** + the nostr.watch ranking, and rebuilds every section: リレーランキング,
  limit パラメータ table, レート制限 table, フィルター値とメッセージサイズ制限 (3 sub-tables),
  時間ベースの制限 (2 sub-tables), 参考文献, バージョン情報.
  Per-relay detail lives ONLY in the evidence files — do NOT add a 詳細分析 / per-relay-prose
  section to the README (the summary tables already link each relay row to its evidence file).
  サポートNIP is NOT tabulated in the README; the 参考文献 nostr.watch line
  (<https://nostr.watch/relays/software>) covers per-implementation NIP support.
  README-ja.md additionally begins with a `## 目次` in-page TOC (level-1 `##` sections only)
  right after the title — preserve/regenerate it. Its リレーランキング anchor embeds the
  Last Checked date (e.g. `#リレーランキング-last-checked-20260626`), so update that one TOC
  link whenever the ranking date changes.
  Cells use current-state facts only — never `changeNote` (learning 8). Preserve the exact
  column headers shown in the current README.

Writers own distinct files, so they never conflict.

---

# Part B — `relay-instances/` NIP-11 research

No clones, no source. Each instance is queried live for its NIP-11 document.

## Instance list (the surveyed relays)

The authoritative list is the table in `relay-instances/README-ja.md`. As of the last run:

```
wss://r.kojira.io                  wss://nostream.ocha.one
wss://relay.damus.io               wss://yabu.me
wss://relay-jp.nostr.wirednet.jp   wss://relay.primal.net
wss://nostr.compile-error.net      wss://relay.snort.social
wss://nrelay-jp.c-stellar.net      wss://snowflare.cc
                                   wss://relay.westernbtc.com
```

Add/remove instances here when the user provides a new list.

## Phase-2 learnings (NIP-11 query)

### 1. Query method
NIP-11 is served over HTTPS with an `Accept` header (swap `wss://` → `https://`):
```bash
curl -s -H "Accept: application/nostr+json" https://<host> | jq '{name, software, version, limitation}'
```
`WebFetch` does NOT work here — it returns the relay's web landing page, not the JSON.
Use `curl` + `jq`.

### 2. Columns to extract
From `.limitation`: `max_message_length`, `max_subscriptions`, `max_limit`,
`max_filters`, `max_subid_length`, `max_event_tags`, `max_content_length`. The
**Software** column = short name from the `.software` repo URL + ` ` + `.version`
(e.g. `git+https://github.com/Cameri/nostream.git` + `1.25.2` → `nostream 1.25.2`).
Use `-` for any field the relay does not publish. Numbers are shown plain (no "KB").

### 3. Unreachable / changed relays
If a relay times out or drops NIP-11, mark its row (e.g. software "—") and note it rather
than deleting the row. Some relays change software/limits between runs — that is exactly
what this survey captures, but the README is current-state only (no diff prose).

### 4. Dates — "Last Checked"
Set the `Last Checked:` line to today (project `CLAUDE.md` → currentDate) on every run,
even if nothing changed — it records when the survey was last verified.

## Phase 3 — query: 1 agent (or 1 per instance)
A single agent can `curl` all ~11 instances (they're cheap). To be robust against one
hanging relay, optionally fan out 1 agent per instance with a short timeout. Each returns
`{ host, software, max_message_length, max_subscriptions, max_limit, max_filters,
max_subid_length, max_event_tags, max_content_length }` (use `-` for missing).

## Phase 4 — write: 2 README writers
One writes `relay-instances/README-ja.md`, one writes `relay-instances/README-en.md`,
each rebuilding the single limitation table (same columns + the `-` note + the top
back/lang link line) and the `Last Checked:` line.

---

## How to run it (the workflows)

Two independent **Workflow** runs. Skeletons:

### Part A — relay/
```js
export const meta = {
  name: 'update-relay-research',
  description: 'Re-research the 6 relay implementations and rewrite evidence + README',
  phases: [
    { title: 'research', detail: '1 agent per relay (6)' },
    { title: 'write-evidence', detail: '1 agent per evidence md (12: ja+en)' },
    { title: 'write-readme', detail: '1 agent per README (ja, en)' },
  ],
}
const SKILL = 'skills/nostr-research-update-relay/SKILL.md'
const RELAYS = ['strfry','nostream','nostr-rs-relay','khatru','haven','wot-relay']

phase('research')
const reports = (await parallel(RELAYS.map(d => () =>
  agent(`Read ${SKILL}. Follow Part A "Phase 3" for relay dir "${d}". Update its clone, record the commit, research all 7 dimensions with version-pinned URLs, return the report.`,
        { label: d, phase: 'research', schema: RELAY_REPORT_SCHEMA })))).filter(Boolean)

phase('write-evidence')
await parallel(reports.flatMap(r => ['ja','en'].map(lang => () =>
  agent(`Read ${SKILL} Part A "Phase 4". Write relay/evidences/${r.dir}${lang==='en'?'-en':''}.md (${lang}) from: ${JSON.stringify(r)}. Match the existing file's layout exactly.`,
        { label: `${r.dir}-${lang}`, phase: 'write-evidence' }))))

phase('write-readme')
await parallel([
  () => agent(`Read ${SKILL} Part A "Phase 4". Rebuild relay/README-ja.md (all sections) from: ${JSON.stringify(reports)}. Also refetch the nostr.watch ranking.`, { label: 'README-ja', phase: 'write-readme' }),
  () => agent(`Read ${SKILL} Part A "Phase 4". Rebuild relay/README-en.md (English mirror) from: ${JSON.stringify(reports)}.`, { label: 'README-en', phase: 'write-readme' }),
])
```

### Part B — relay-instances/
```js
export const meta = {
  name: 'update-relay-instances',
  description: 'Re-query NIP-11 for each relay instance and rewrite the README tables',
  phases: [
    { title: 'query', detail: '1 agent per instance' },
    { title: 'write-readme', detail: '1 agent per README (ja, en)' },
  ],
}
const SKILL = 'skills/nostr-research-update-relay/SKILL.md'
const INSTANCES = ['r.kojira.io','relay.damus.io','relay-jp.nostr.wirednet.jp','nostr.compile-error.net','nrelay-jp.c-stellar.net','nostream.ocha.one','yabu.me','relay.primal.net','relay.snort.social','snowflare.cc','relay.westernbtc.com']

phase('query')
const rows = (await parallel(INSTANCES.map(h => () =>
  agent(`Read ${SKILL} Part B. curl NIP-11 for https://${h} and return the row fields.`,
        { label: h, phase: 'query', schema: INSTANCE_ROW_SCHEMA })))).filter(Boolean)

phase('write-readme')
await parallel([
  () => agent(`Read ${SKILL} Part B "Phase 4". Rebuild relay-instances/README-ja.md table from: ${JSON.stringify(rows)}`, { label: 'README-ja', phase: 'write-readme' }),
  () => agent(`Read ${SKILL} Part B "Phase 4". Rebuild relay-instances/README-en.md table (English) from: ${JSON.stringify(rows)}`, { label: 'README-en', phase: 'write-readme' }),
])
```

## After the workflow
- `git diff --stat relay/ relay-instances/` to review scope.
- Part A: spot-check that 2-3 reference URLs use the NEW commit hash and resolve to the
  cited line. Confirm "バージョン情報" hashes match the evidence "確認バージョン".
- Commit per the project commit rules (Japanese message; do not `git add -A`).
