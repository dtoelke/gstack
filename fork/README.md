# This fork

`dtoelke/gstack` is [garrytan/gstack](https://github.com/garrytan/gstack) plus a
few fixes that have not landed upstream. An install that follows it gets every
upstream release with those fixes already merged in, so the install itself
never carries a local edit.

## What the fork adds

Everything the fork adds is a commit on `main`; `git log --first-parent
--no-merges upstream/main..main` lists them.

- **Pre-gate fixes** (`lib/pregate/`, `docs/pregate.md`, their tests):
  - literals: a lowercase name counts a test as a consumer only when the test
    names the changed module; an uppercase constant only when a JS/TS test
    names the constant or imports the module.
  - lane selection: a removed non-test path that no remaining test selects
    owes no lane.
  - lanes: a selected vitest file runs under the first script-named config
    that includes it; the `vitest_scripts` config key limits which scripts'
    configs run.
  - supervise: the stall guard counts CPU across the runner's process tree.
- **Fork wiring** (`bin/gstack-update-check`, `fork/`): the version check
  reads this fork instead of upstream, and merges upstream into the fork
  first.

## How an install follows it

The install is a plain checkout of this fork: `origin` is
`https://github.com/dtoelke/gstack.git`, branch `main`. Nothing else on the
machine is configured.

Each time `bin/gstack-update-check` goes to the network (at most once an hour)
it runs `fork/sync-upstream`, which asks GitHub to merge `garrytan/gstack`
`main` into this fork's `main`. The check then compares the install with the
fork, and `/gstack-upgrade` fast-forwards the install to it. The merge needs a
`gh` login with push access to the fork; without one the check still works and
the fork just stops advancing.

## Changing what the fork adds

Never edit the files of an install. A tracked edit there is shelved by the
updater the first time one file conflicts with a release, and it blocks
`bin/gstack-upgrade`.

1. Clone the fork, commit the change on `main` and push it.
2. The install picks it up at its next version check, like any release.

A fix that upstream would take belongs in a pull request to `garrytan/gstack`
as well, from a branch cut from upstream's `main`. Once upstream merges it the
fork's copy stops being a difference.

## When the sync conflicts

The version check prints `FORK_SYNC_CONFLICT ...` and the install stays on the
version it has, patched and working. To resolve:

```sh
git clone https://github.com/dtoelke/gstack.git && cd gstack
git remote add upstream https://github.com/garrytan/gstack.git
git fetch upstream main
git merge upstream/main          # fix the conflicts, then git commit
bun install --frozen-lockfile
bun test test/pregate-import-graph-paths.test.ts test/pregate-policy-lane-verdicts.test.ts \
  test/pregate-removed-paths.test.ts test/pregate-selection.test.ts test/pregate-supervise.test.ts
git push origin main
```

Compare the test failures with the same run on `upstream/main`: only a test
that fails on the merge and passes upstream is the merge's doing.

`FORK_SYNC_FAILED ...` means GitHub refused the merge call; the line carries
GitHub's reason (a login without push access to the fork, or a token without
the `workflow` scope when upstream changed a workflow file).
