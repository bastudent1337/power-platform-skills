# Wave 3 Handoff — Power Platform Skills Consolidation Campaign

**Read this whole file before doing anything.** It's written to be self-contained: a fresh session with no memory of prior conversations should be able to pick up exactly where this left off. If anything here conflicts with what you find live in the repos, trust the repos — this file can go stale.

## Why this file exists

The prior session hit a persistent bug: `add_repo` for 4 newly-forked GitHub repos kept failing with "MCP tool call requires approval" and wouldn't clear on retry, mid- or start-of-session. The operator asked for a handoff so they could start a **new chat with those repos attached as sources at session-creation time** instead (a different code path than mid-session `add_repo`, which may sidestep the bug). This file is that handoff. It lives on `power-platform-skills`' designated working branch (`claude/power-platform-skills-merge-6a56rx`, reset from `ops`) rather than `main`, because `main` on this fork is a **deliberately byte-pristine mirror** of `microsoft/power-platform-skills` — nothing but upstream commits ever land there. Do not push anything to this fork's `main`, ever, for any reason. This file itself should probably not be merged into `ops` either (`ops`'s documented job is carrying `sync-upstream.yml`, nothing else) — treat this branch as scratch/reference, not something to land anywhere.

## Sources this new session needs attached

The actual work happens in **`bastudent1337/claude-skills`** (private) — that repo MUST be a source or nothing here is actionable. Recommended full set:

| Repo | Role | Note |
|---|---|---|
| `bastudent1337/claude-skills` | **The work** — private skill library, all campaign state lives here | Default branch `main` |
| `bastudent1337/power-platform-skills` | Already-wired vendor mirror + `ops`-branch sync workflow — reference/template for wiring the new forks | This repo; `main` pristine, `ops` carries the workflow |
| `bastudent1337/power-cat-skills` | **New fork**, needs wiring | Default branch `main` |
| `bastudent1337/Dataverse-skills` | **New fork**, needs wiring | Default branch `main` |
| `bastudent1337/List-Formatting` | **New fork**, needs wiring | ⚠️ **Default branch `master`, not `main`** — will trip up anything that assumes `main` uniformly |
| `bastudent1337/fluentui-system-icons` | **New fork**, needs wiring | Default branch `main` |

`bastudent1337/power-platform-solution-blueprint` (PPSB) is also relevant (see below) but was already forked and wired in an earlier wave and shouldn't need re-attaching if the new session's scope already includes it; add it if not.

## Campaign state as of 2026-07-11 (all merged, verify via `git log` on each repo before trusting)

This is "PP Skills v2" — the user's own library (`claude-skills`) collided by name with Microsoft's official plugins, and they wanted (a) Microsoft's upstream content vendored in for reference/monitoring, (b) their own PP-domain skills renamed into an unambiguous `bauer-` namespace with an explicit "we override vendor defaults" contract, and (c) durable tracking of upstream changes going forward. Shipped so far, in order:

1. **PR #107** (v1 foundation): `microsoft/power-platform-skills` vendored read-only at `claude-skills:vendor/power-platform-skills/` via `git subtree add --squash`, pinned at `eec8ece`. Public fork `bastudent1337/power-platform-skills` set up as the monitoring scope-bridge (main pristine/FF-only from upstream; `ops` branch carries `.github/workflows/sync-upstream.yml`, dispatch-first). `vendor/vendor-map.json` + `vendor/VENDOR-MAP.md` created as the provenance ledger. `docs/upstream-digest-playbook.md` created as the weekly-digest runbook.
2. **PR #109** (Wave 0): vendor-map v2 — wired in `ppsb` (PPSB, `sabrish/power-platform-solution-blueprint`, fork already existed: `bastudent1337/power-platform-solution-blueprint`), added `list-formatting`/`fluentui-system-icons` source entries, fixed a mis-attribution (`bauer-sharepoint-json-formatting`'s bundled samples are from `pnp/List-Formatting`, not power-cat-skills as originally recorded), added file-level-port rows for `bauer-solution-parser` (dual upstream: PPSB + power-cat-skills) and `bauer-icon-library` (fluentui-system-icons).
3. **PR #112** (Waves 1+2): the big rename — all 36 PP-domain skills renamed to `bauer-*` (e.g. `canvas-apps`→`bauer-canvas-apps`; exceptions `power-platform-team`→`bauer-platform-team`, `plugins`→`bauer-dataverse-plugins`) to kill collisions with installed vendor plugins. Every renamed skill gained an **overlay contract**: a vendor-canon header (origin + pin state), a precedence line ("this skill wins for Bauer work"), and a placeholder `## Overrides vs vendor defaults` section — **this placeholder is what Wave 3 fills in**. The 2 verbatim-vendored skills (`add-data-source`, `configure-canvas-mcp`) were dropped in favor of routing to the vendor plugin directly. `MARKETPLACE_VERSION` bumped 2.0.0→3.0.0 (breaking `/plugin:skill` names — installed marketplaces/Cowork bundles needed reinstall). New CI gate `check_version_coupling.py`: any skill `metadata.version` bump requires a `MARKETPLACE_VERSION` bump in the same change, or the build fails.
4. **PR #114** (Wave 4): the weekly digest switched to **issue-mode** — bucket-B/C findings that aren't a clean auto-resync now file (or update, deduped by title prefix) a tracked GitHub issue in `claude-skills` instead of only living as PR prose. `docs/upstream-digest-playbook.md` Step 0 was genericized to loop over every `vendor-map.json` source rather than naming two by hand, and gained a `fork-exists-unwired` status (forked, no `ops` sync workflow yet — `ppsb`'s state) distinct from `fork-pending`.

**Outstanding / known gaps — read before assuming anything is done:**
- The weekly digest Routine **does not currently exist**. It was deleted (its prompt needed updating for issue-mode and Routine prompts can't be edited in place) and the recreate call was explicitly denied by the operator mid-session — not a bug, a deliberate pause. Don't recreate it unless the operator asks; if they do, the exact prompt design is in the Wave 4 section of `/root/.claude/plans/alright-so-i-did-calm-coral.md` in the prior session (not available to you) — reconstruct from `docs/upstream-digest-playbook.md`'s own content instead, which is self-sufficient.
- No GitHub MCP tool available in the prior session could create repo labels (only read one). The 4 labels the issue-mode playbook references (`upstream-watch`, `bucket-B`, `bucket-C`, `needs-operator`) don't exist yet in `claude-skills`. If your session has a working label-create tool, creating them would help; if not, it's a manual operator follow-up, not a blocker.
- `claude-skills`' `main` moves fast — other work has landed concurrently on it three separate times during this campaign so far (PRs #108, #110, #111, #113 all landed mid-wave). Always re-fetch `origin/main` immediately before pushing anything and be ready to merge + resolve. Two patterns already seen and resolved: (a) two PRs independently regenerating the same archivist build artifact (`skills/bauer-skill-finder/assets/Skill-Navigator.html`/`-SPO.html`) — resolution is to regenerate fresh via `python3 skills/bauer-skill-archivist/references/build_skill_navigator.py` (with `BAUER_SKILLS_DIR`/`BAUER_NAVIGATOR_OUT_DIR` env vars pointed at the real checkout and a temp dir — see `.github/workflows/sync-and-publish.yml` for the exact invocation), never hand-resolve conflict markers in that generated blob (there's a hard rule in `bauer-skill-finder/SKILL.md` forbidding manual regen of these assets by anything except the archivist scripts); (b) two PRs editing adjacent rows of the alphabetically-sorted bookkeeping table `skills/bauer-skill-improver/references/upgrade-coverage.md` — a real two-sided conflict, resolved by combining both sides' rows (git's 3-way merge just can't tell two independent adjacent-row edits apart).

## Current `vendor/vendor-map.json` `sources` block (as of `main` @ `f8fa965`, verify it's still current)

```json
{
  "power-platform-skills": {"upstream": "microsoft/power-platform-skills", "fork": "bastudent1337/power-platform-skills", "local_vendor_path": "vendor/power-platform-skills/", "pin": "eec8eceb0888e1891879ff37f56caceadac71748", "pin_date": "2026-07-10", "sync": "subtree-squash"},
  "power-cat-skills": {"upstream": "microsoft/power-cat-skills", "fork": null, "local_vendor_path": null, "pin": null, "pin_date": null, "sync": null, "status": "fork-pending"},
  "dataverse-skills": {"upstream": "microsoft/Dataverse-skills", "fork": null, "local_vendor_path": null, "pin": null, "pin_date": null, "sync": null, "status": "fork-pending"},
  "ppsb": {"upstream": "sabrish/power-platform-solution-blueprint", "fork": "bastudent1337/power-platform-solution-blueprint", "local_vendor_path": null, "pin": null, "pin_date": null, "sync": null, "status": "fork-exists-unwired"},
  "list-formatting": {"upstream": "pnp/List-Formatting", "fork": null, "local_vendor_path": null, "pin": null, "pin_date": null, "sync": null, "status": "fork-pending"},
  "fluentui-system-icons": {"upstream": "microsoft/fluentui-system-icons", "fork": null, "local_vendor_path": null, "pin": null, "pin_date": null, "sync": null, "status": "fork-pending"}
}
```

**The 4 new forks now exist** (verified by the operator and confirmed via `search_repositories` in the prior session):
- `bastudent1337/power-cat-skills` (default branch `main`)
- `bastudent1337/Dataverse-skills` (default branch `main`)
- `bastudent1337/List-Formatting` (default branch **`master`**)
- `bastudent1337/fluentui-system-icons` (default branch `main`)

So `power-cat-skills`, `dataverse-skills`, `list-formatting`, `fluentui-system-icons` all need their `status` flipped away from `"fork-pending"` once wired (see Task 1 below), and `ppsb` needs its `ops` workflow added to move it from `"fork-exists-unwired"` to fully wired.

## Wave 3 task list

### Task 1 — Wire each new fork's `ops`-branch sync workflow

Template: this repo (`power-platform-skills`)'s `ops` branch, file `.github/workflows/sync-upstream.yml`. Read it first. For each of the 4 new forks (and `ppsb`, which is still unwired):
1. Create an `ops` branch at the fork's current default-branch tip.
2. Add `.github/workflows/sync-upstream.yml` adapted to that fork: `workflow_dispatch`-first (never rely on `schedule:` alone — same reasoning as the original: avoids the 60-day auto-disable on inactive-default-branch scheduled workflows), the sync step is `gh repo sync <owner>/<repo> --source <upstream-owner>/<upstream-repo> --branch <default-branch>`. **For `List-Formatting`, `--branch master`, not `main`** — check every fork's actual default branch before assuming `main`.
3. Push the `ops` branch. No PR needed against the fork's `main` (never touch fork `main`s — same pristine-mirror invariant as this repo).
4. Confirm the workflow runs once (manual dispatch) before moving on.

### Task 2 — Vendor-pin size gate, then subtree-vendor what's small enough

Clone each new fork with `--depth 1`, check size (`du -sh`). If small enough (rough heuristic used earlier in this campaign: comparable to or smaller than `power-platform-skills`' own ~13MB vendored tree — judgment call, don't over-engineer this threshold):
- `git subtree add --prefix=vendor/power-cat-skills <local-clone-path> main --squash` (adjust branch name per fork) in `claude-skills`.
- Same for `dataverse-skills` if small enough.
- **Never vendor `list-formatting` or `fluentui-system-icons` as subtrees** — this was an explicit decision in Wave 0 (they're asset-heavy sample/icon libraries, not skill-content repos; fork+watch only, no local vendor copy needed for the reconcile below since only specific files/paths matter, not the whole tree).

Update `vendor/vendor-map.json`'s `sources` block for every source touched: `fork` (URL), `local_vendor_path` (if vendored), `pin` + `pin_date` (if vendored), `sync: "subtree-squash"` (if vendored), and flip `status` away from `fork-pending`/`fork-exists-unwired` to reflect the new wired state (drop the `status` key entirely once a source is fully wired and vendored, matching how `power-platform-skills`'s own entry has no `status` key today).

### Task 3 — The per-skill triage-reconcile pipeline (the actual point of Wave 3)

This is the reason Wave 3 exists: **~34 advisory skills** in `claude-skills` (`bauer-dataverse`, `bauer-alm`, `bauer-licensing`, `bauer-governance`, `bauer-security`, `bauer-testing`, `bauer-sharepoint*`, etc. — grep `vendor/vendor-map.json` for `"policy": "none"` rows with a non-null `origin.repo`) were written against Microsoft's power-cat-skills / Dataverse-skills content **without ever being able to diff against the real thing** — their `vendor-map.json` rows are all `confidence: "inferred"`. Now that the forks exist, actually check.

Per advisory skill:
1. **Collect** (cheap model — haiku): find candidate upstream base file(s) in the now-vendored/cloned `power-cat-skills` (or `dataverse-skills`) tree that plausibly correspond to this Bauer skill's topic. Power-cat-skills' own layout may not map 1:1 to Bauer's skill names — this is real archival work, not a lookup. **"No base found" is a completely valid, expected outcome for some skills** — don't force a match.
2. **Diff + classify** (sonnet): compare the Bauer skill's current body against whatever upstream file(s) were found. Classify: **ingest-now** (upstream has material worth pulling in that Bauer currently lacks), **watch-only** (Bauer's version already supersedes or diverges deliberately — most common expected outcome), or **no-base-found** (flip that row's `origin` to `null`, `confidence: "verified"` — this itself is a valuable, permanent finding, not a failure). Draft the skill's `## Overrides vs vendor defaults` section content either way — even a "no meaningful overlap with vendor content found" is worth stating explicitly rather than leaving the placeholder text in forever.
3. **Adjudicate** (opus or equivalent top-tier model — this repo's own convention for judgment-heavy roles): review every ingest-now proposal and any drastic divergence. Approved deltas get folded into the Bauer skill's body, its `metadata.version` bumped (which — per the version-coupling gate — requires a `MARKETPLACE_VERSION` bump in the same change), and the vendor-map row updated to `confidence: "verified"` with real `base_version`/`base_pin` values.
4. **Priority**: definitively resolve the **9 "weak inference" rows** first — these were flagged in earlier waves as possibly Bauer-original rather than actually power-cat-derived, and this fork finally makes that answerable: `bauer-sharepoint`, `bauer-sharepoint-api`, `bauer-sharepoint-json-formatting` (upstream is actually `list-formatting`, already corrected — verify its samples/catalog are current), `bauer-sharepoint-spfx`, `bauer-m365-integration`, `bauer-env-strategy`, `bauer-backup-strategy`, `bauer-observability`, `bauer-integration-patterns`.

### Task 4 — Ship

Full local gate battery before every push (this campaign's proven battery — run all of these, all must pass):
```
python3 skills/bauer-skill-improver/references/quality_review.py --skills-path skills --output-dir <scratch-dir>   # Errors: 0
python3 skills/bauer-skill-improver/scripts/test_extract_yaml.py
python3 skills/bauer-skill-improver/scripts/test_hard_rule_buckets.py
python3 skills/bauer-skill-archivist/references/build_marketplace.py --skills-dir skills --archive-root .   # must print PARTITION OK
python3 skills/bauer-skill-archivist/references/build_agents.py --kit-dir claude-code-setup --archive-root .
python3 skills/bauer-skill-archivist/references/sync_setup_kit.py --check
python3 skills/bauer-skill-archivist/references/assert_layout.py --archive-root .
python3 skills/bauer-skill-improver/scripts/lint_hardcoded_rates.py --skills-path skills
python3 skills/bauer-skill-improver/references/quality_review.py --lint-logs skills/bauer-skill-improver/references/logs
python3 skills/bauer-skill-improver/references/quality_review.py --lint-scripts skills
python3 skills/bauer-brand-builder/scripts/sync-runtime-to-standard.py --check
python3 skills/bauer-skill-improver/scripts/refresh_bookkeeping.py --all --check   # or --write then re-check
python3 skills/bauer-skill-archivist/references/check_version_coupling.py --base origin/main
```
Plus a repo-wide grep to confirm no leftover old (pre-rename) bare slugs got reintroduced by copy-pasted vendor content (the rename campaign's own gate — anchored kebab-token search for `/canvas-apps`, `skills/canvas-apps/`, etc. against the OLD names, expect zero outside `vendor/`).

Re-fetch `origin/main` in `claude-skills` immediately before pushing (see "known gaps" above re: how often it's moved this campaign). Commit, push to a fresh branch off current `main`, open a PR, subscribe to its activity. Consider whether this ships as one PR or several (per-source, or per-skill-batch) — the operator hasn't ruled on that; ask if unclear rather than assuming.

## Where to look for more detail

- `claude-skills:vendor/VENDOR-MAP.md` — the human-readable provenance-ledger doc (schema, standing rules, licensing notes).
- `claude-skills:docs/upstream-digest-playbook.md` — the weekly-digest runbook (self-contained, describes the A/B/C bucket system this Wave partially reuses).
- `claude-skills:DECISIONS.md` — search for the `2026-07-10` and `2026-07-11` rows; each campaign wave has one, in order, with full rationale.
- `claude-skills:skills/power-platform-team/references/vendor-plugins.md` — the "vendor canon wins" doctrine (executor vs. overlay) that motivated the whole rename.
