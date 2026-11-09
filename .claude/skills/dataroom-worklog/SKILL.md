---
name: dataroom-worklog
description: Records a finished nicegram-desktop task in the Nicegram dataroom work log, as the dataroom protocol block at the top of AGENTS.md requires, and finds merges to dev that no work log entry names yet. Use when a task's change has reached dev or has an open pull request, after the commit that ends a task, when the user asks to record work in the dataroom or the work log, or at session start to check for uncovered commits.
---

# Dataroom Work Log (nicegram-desktop)

The protocol block at the top of `AGENTS.md` is the rule; this skill is how to keep it in
this repository. The dataroom is the private repository `nicegram/dataroom`; its
`docs/work-log/README.md` is the authority on the entry schema.

Nothing writes these entries automatically. The dataroom's nightly audit only lists the
`dev` commits that no entry names (`docs/repositories/STATUS.md`, column *Uncovered
commits*). An entry written by the agent that did the work is the only record of what was
done and why, so **a task is not finished until its entry is merged** — or until the user has
been told why it is not.

A fork of `telegramdesktop/tdesktop`; `dev` is the default branch. A sync with upstream is a
task of its own and gets its own entry, naming the merge that brought it in.

## 0. The dataroom checkout

Use a local checkout of `git@github.com:nicegram/dataroom.git`. If you do not know where it
is, ask the user, or clone it next to this repository, never inside it. Every command below
that is not prefixed with `git -C` runs inside that checkout (`$DATAROOM`); `$REPO` is this
repository's checkout.

## 1. Before the task

```bash
git -C "$DATAROOM" pull --ff-only
git -C "$DATAROOM" rev-parse HEAD     # keep it: this is the entry's dataroom_read
```

- Read this repository's row (`github:nicegram/nicegram-desktop`) in
  `docs/repositories/registry.json` and the profile under `docs/repositories/` it names, if
  any; for a user-visible feature also `docs/kb/README.md` and the feature's record in
  `docs/kb/features/`.
- **Jira key**: from the commit subject (`[NCG-XXXXX] ...` or `NCG-XXXXX ...`) or the branch
  name (`[A-Z]+-\d+`); a task without one has none.
- **OKR id**: only one the task really has in `docs/okr/governance.json`.
- Never invent a Jira key or an OKR id; leave the list empty instead.

## 2. When to write the entry

One entry per task, not per commit, as soon as the change has a commit on `dev` or an
open pull request:

| How it landed | `ref` |
|---|---|
| Merged pull request | `--pull-request <url>` and `--commit <merge commit>` |
| Direct push to `dev` | `--commit <sha>`; a merge commit that carried it goes in `ref.commits` |
| Several pull requests or commits for one task | the last one in `--pull-request` / `--commit`, the others in `ref.pull_requests` / `ref.commits` |
| Same task in another repository too | one entry per repository, same slug with the repository's name in it |

## 3. Write it

```bash
git switch -c worklog/$(date -u +%F)-nicegram-desktop-<slug> origin/main --no-track
f=docs/work-log/$(date -u +%Y/%m)/$(date -u +%F)-nicegram-desktop-<slug>.json
python3 scripts/agentops/worklog.py new --repository github:nicegram/nicegram-desktop \
  --slug nicegram-desktop-<slug> \
  --summary "<what changed and why: 10 to 600 characters, English, no em dash>" \
  --pull-request https://github.com/nicegram/nicegram-desktop/pull/<n> --commit <merge sha> \
  --branch <branch> --dataroom-read <sha from step 1> --jira NCG-XXXXX > "$f"
```

- **Always pass `--dataroom-read`.** Without it `new` takes the checkout's current `HEAD`,
  which is not the commit you read if you pulled since. If you cannot know it (a backfill),
  set `"dataroom_read": null` and add `"dataroom_read_gap": "<why>"` to the file by hand.
- **Features.** A user-visible feature appeared, changed or was removed → in the same dataroom
  pull request update `docs/kb/features/<id>.json` and `docs/kb/changes.jsonl`, run
  `python3 scripts/kb/build_kb.py` and `python3 scripts/kb/agent_gate.py`, and pass
  `--feature <id> --kb-ref <dataroom pull request url>`. A bug fix that restores documented
  behaviour, hardening or an internal change is not a feature change: say "no KB change" in
  the summary.
- **Summary.** Name the screens, settings, API or TL schema methods, or build files the change
  touched and the effect, the way the pull request title would explain it to someone
  outside this repository.

## 4. Check and land

```bash
python3 scripts/agentops/worklog.py --check
bash scripts/check-docs.sh
git add "$f"
git commit -m "work log: nicegram-desktop <what changed> (#<n>)"
git push -u origin HEAD
gh pr create -R nicegram/dataroom --base main \
  --title "Work log: nicegram-desktop <what changed>" --body "<entry id, checks run>"
gh pr merge <dataroom pr> -R nicegram/dataroom --merge
git switch main && git pull --ff-only
```

- The commit message follows the dataroom's style (`work log: ...`), not this repository's
  commit subjects.
- A merged entry is never edited or deleted. A mistake is fixed by a new entry whose
  `"corrects"` names the wrong one.

## 5. Find uncovered merges

Run this at session start, and before you call a task finished. It is the nightly audit's
own code (`scripts/agentops/registry.py`) applied to the current state, so it also sees
entries merged since last night:

```bash
git -C "$REPO" fetch origin dev
python3 - "$REPO" <<'EOF'
import glob, json, pathlib, sys
sys.path.insert(0, 'scripts/agentops')
import registry
repo = 'github:nicegram/nicegram-desktop'
row = next(r for r in registry.load()['repositories'] if r['id'] == repo)
since = (row.get('seen') or {}).get('adopted_commit')
if not since:
    sys.exit('no adopted_commit yet: the nightly audit records it the first night the block is on dev')
entries = [json.load(open(f)) for f in glob.glob('docs/work-log/**/*.json', recursive=True)]
shas, prs = registry.covered_by(entries, repo)
print(registry.uncovered(pathlib.Path(sys.argv[1]), 'origin/dev', since, shas, prs))
EOF
```

A commit counts as covered when an entry names it, names the pull request in its merge
subject, or names any commit a merge brings in.

For every commit it prints:

- **The task was yours in this session** → write its entry now (steps 3–4).
- **Someone else's work** → tell the user which commits and pull requests lack an entry and
  who wrote them. Do not write an entry for another person's task without the user's
  go-ahead; when they give it, say in the summary who did the work and who merged it.

## Output

Report the entry id, the dataroom pull request and its state, and what step 5 still lists.
If the dataroom could not be reached, say so and leave the user the commands to run.
