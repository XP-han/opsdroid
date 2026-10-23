# Implementation review — upstream PR #2079

**Deliverable 8 (added)** · Reviewer: Chenyan Zhou (u8071817)
**Closes the outstanding item** "re-run my reproduction against the final implementation"
from the work-package index (held by the group) — for this PR.

| | |
|---|---|
| PR | <https://github.com/opsdroid/opsdroid/pull/2079> — *"Replace pkg_resources with importlib.metadata"* |
| Author | `hkwongax` |
| Commit | `d4ef713` (2026-10-15) |
| Base | `opsdroid:main` |
| Diff | 2 files, **+8 / −10** (`opsdroid/loader.py`, `tests/test_loader.py`) |
| State at review | **open**, not merged, not a draft, no reviewers requested yet |
| Reviewed tree | `a5/review/pr2079` — a `git archive` of `d4ef713`, so the review is of exactly what the PR contains |

> **Who is `hkwongax`?** I cannot map that GitHub account to a group member from here,
> so I did not assume it is Linrui. The verification below is valid for *any* candidate
> fix, and the same command should be re-run against the group's own branch if it
> differs. Nothing in this review creates, edits, comments on or pushes to that PR.

---

## 1. Conformance with the contract I gave Linrui

[`04-fix-interface-contract.md`](04-fix-interface-contract.md) lists seven hard
requirements. Measured against the diff:

| # | Requirement | Status in PR #2079 |
|---|---|---|
| R1 | Importing `opsdroid.loader` must not import `pkg_resources` | **met** — `from importlib.metadata import entry_points`; `grep pkg_resources` over the shipped package returns nothing |
| R2 | Still enumerate by group `opsdroid_{modules_type}s` | **met** — `entry_points(group=epname)`, `epname` construction untouched |
| R3 | Still build the `name -> entry point` mapping | **met** — `{ep.name: ep for ep in ...}`, only the local variable was renamed |
| R4 | Still call `.load()` and use its return value as the module | **met** — `loader.py:86` untouched |
| R5 | One code path for 3.11/3.12/3.13 | **met** — no version branching; verified on all three (see §3) |
| R6 | Fresh venv on 3.13 must work with no pin and no manual step | **met** — see §2 |
| R7 | `tests/test_loader.py` must import without `pkg_resources` | **met** — `from importlib.metadata import EntryPoint`, `EntryPoint("myep", "os.path", "opsdroid_databases")` |

It also avoids every trap the contract listed:

* **The `UnboundLocalError` pitfall is avoided.** The PR renames the local variable to
  `available_entry_points` rather than shadowing the imported `entry_points`. This is the
  exact failure I reproduced in `J-313-poc-naive`
  (`../experiments/logs/J-313-poc-naive.log`), and the PR does not walk into it.
* **No upper bound was added to setuptools**, so it is a fix rather than a countdown.
* **The `opsdroid_parserss` group typo was left alone**, keeping the change
  behaviour-preserving as requested.
* **`opsdroid/core.py` was not touched**, so the `module.__dict__` contract is unchanged.
* **The test's mock target matches the new import** — `mock.patch("opsdroid.loader.entry_points")`
  resolves, because the name is now a module-level attribute of `opsdroid.loader`.

---

## 2. End-to-end verification against my reproduction

Command (the verifier I built for exactly this purpose):

```bash
cd /Users/comp6120/a5/experiments
bash verify_fix.sh /Users/comp6120/a5/review/pr2079
```

Full transcript: `../experiments/logs/VERIFY-pr2079.log`. **Exit code 0 — 9 passed,
0 failed.** The load-bearing lines:

```
[info] interpreter: Python 3.13.16
[info] setuptools installed: 84.0.0
[PASS] pkg_resources is NOT importable - same precondition as the issue report
[PASS] opsdroid.loader imports without pkg_resources
[PASS] 'opsdroid version' -> opsdroid 0+unknown
[PASS] 'python -m opsdroid' reaches the CLI (the traceback in the issue is gone)
[PASS] 'opsdroid version' still works with setuptools uninstalled and no pin
[PASS] 'opsdroid start' loaded both entry-point modules and kept running
[PASS] debug log confirms the entry point was used: fixtureconn
[PASS] debug log confirms the entry point was used: fixtureskill
DEBUG opsdroid.loader Found installed package for skill 'fixtureskill' support.
DEBUG opsdroid.loader Found installed package for connector 'fixtureconn' support.
```

Two things make this more than "it starts":

1. **The negative control passed.** The venv really had setuptools 84.0.0 and no
   `pkg_resources`, so this is the environment that fails on `main` — not a
   coincidentally-working one.
2. **Entry-point discovery is proven, not assumed.** The `fixtureconn` connector and
   `fixtureskill` skill exist only as entry points in an installed wheel; the debug lines
   are opsdroid itself confirming it found and used them. That exercises R2, R3 and R4
   together through the real code path.

## 3. opsdroid's own test suite

Transcript: `../experiments/logs/VERIFY-pr2079-tests.log`. The PR tree installed
editable, `setuptools` forced to `>= 82`:

| Python | setuptools | `pytest tests/test_loader.py` |
|---|---|---|
| 3.13.16 | 84.0.0 | **48 passed** |
| 3.12.15 | 84.0.0 | **48 passed** |
| 3.11.17 | 84.0.0 | **48 passed** |

For comparison, on unpatched `main` the same command cannot even **collect** the file
under setuptools 84 (`../experiments/logs/TESTSUITE-evidence.log`): 48 passed with
setuptools 81, collection error with 84, and still a collection error after patching only
`loader.py`. The PR closes that gap because it changes both files — which is exactly the
point I flagged to Weixin.

---

## 4. Small gaps against the contract (none blocking)

These are refinements, not defects. I am recording them so the group can decide whether
to raise them with the author rather than discovering them in review.

1. **The ported test does not assert the group.** My T2 suggested
   `mock_entry_points.assert_called_once_with(group="opsdroid_databases")`. The PR keeps
   the original assertion (`loaded[0]["config"]["name"] == "myep"`), so a future edit that
   passes the wrong group — or a misspelled one — would not be caught. Cheap to add.
2. **The ported test does not assert that `.load()` was used.** The original test did not
   either, so this is not a regression; but R4 is the requirement most likely to break
   silently, and `self.assertIs(loaded[0]["module"], os.path)` would pin it down.
3. **`setuptools>=76.0.0` is still in `install_requires`** (`setup.cfg:69`). It is now
   dead weight: after this PR nothing in the shipped package imports setuptools, and I
   verified the patched code runs with setuptools *uninstalled entirely*. Dropping it is
   optional and would slightly widen the PR; if it is not dropped, a comment in the PR
   description explaining why would prevent it looking like an oversight.
4. **No changelog / release note.** The repo has a release-drafter workflow, so a PR
   title prefixed appropriately may be enough, but it is worth checking the
   `docs/maintainers` conventions before merge.

I deliberately did **not** send any of this to the PR author: my brief forbids contacting
maintainers and reviewers. Handing it to the group is the intended route.

---

## 5. What the PR does not have to worry about (and why)

* **Ordering / duplicate entry-point names.** I measured both APIs returning the same
  names in the same order, with the same winner in the dict comprehension, and verified a
  same-name shadow distribution adds nothing under either API. See
  `../experiments/ENTRYPOINT-API-COMPARISON.md` §1, §3, §6b.
* **The late-`sys.path` difference.** `importlib.metadata` *does* see a directory appended
  after the import where `pkg_resources` did not (`loader.py:451` → `:456`). This is a
  behaviour change but a benign improvement, and it is not a reason to block the PR. It
  would be worth one sentence in the PR description so maintainers are not surprised
  later.
* **Extras in entry-point values.** opsdroid never uses them, and the API comparison
  confirms it, so the loss of `pkg_resources`'s extras check on `.load()` is irrelevant.

---

## 6. Verdict

**The PR fixes the bug I reproduced, on every supported Python version, in exactly the
environment that fails today, without a pin, and with the entry-point path proven to work
end to end.** It satisfies all seven contract requirements and avoids the shadowing
pitfall.

From my work package, this closes the "review the final implementation" obligation for
PR #2079. The open questions are bookkeeping, not code:

* confirm `hkwongax` is our group's author (or re-run `verify_fix.sh` on our branch);
* decide whether to ask for the two cheap test assertions in §4;
* decide whether to drop `setuptools>=76.0.0` in a follow-up.

Re-run the check at any time — it is deterministic and takes about a minute:

```bash
cd /Users/comp6120/a5/experiments
bash verify_fix.sh /Users/comp6120/a5/review/pr2079   # exit 0
bash verify_fix.sh /Users/comp6120/a5/upstream-opsdroid  # exit 1, still broken on main
```
