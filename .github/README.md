# Why `automation` is this fork's default branch

This is a fork of [jherrma/supercharged-memory](https://github.com/jherrma/supercharged-memory).
`master` here is meant to be a **byte-identical mirror** of upstream's `master`, and
[`workflows/sync-upstream.yml`](workflows/sync-upstream.yml) keeps it that way once a day.

The workflow cannot live on `master`. GitHub fires `schedule` events **only from the default
branch**, and a workflow file committed to `master` would put this fork one commit ahead of
upstream — after which no fast-forward ever succeeds again. So the workflow lives on this
branch, and this branch is the default so that the schedule runs.

That is the whole reason. Nothing else here depends on which branch is default.

## What each branch is

| Branch       | What it holds                                                        |
| ------------ | -------------------------------------------------------------------- |
| `master`     | upstream's history, nothing of ours — fast-forwarded daily            |
| `automation` | this file, the workflow, and a snapshot of `master` from the day it was branched |
| anything else | work in progress, headed for a pull request against upstream          |

`automation` deliberately does **not** track upstream. Nothing in the workflow reads the
repository's own code, so a stale snapshot costs nothing — but do not read it as a second mirror.

## Running it by hand

Actions → **sync-upstream** → *Run workflow*. Or:

```bash
gh workflow run sync-upstream.yml --repo Syndic0r/supercharged-memory --ref automation
```

A one-off sync needs none of this — `gh repo sync Syndic0r/supercharged-memory --branch master`
does the same fast-forward server-side.

## When it goes red

One cause: something was committed to this fork's `master`, so it no longer fast-forwards. The job
summary says so. Move that commit to a branch of its own and re-run — the workflow will not paper
over the divergence with a merge, because a merge makes it permanent and turns every later sync
into a merge too.

## The 60-day clock

A public repository's scheduled workflows are **disabled automatically after 60 days with no
repository activity**, and a workflow *run* does not count — a commit does. The last step of the
workflow therefore writes a timestamp to `.github/last-sync-run` on this branch each time it runs,
which keeps the clock from expiring during a quiet stretch upstream. It never commits to `master`.

If the schedule is disabled anyway, GitHub emails the repository owner and the Actions tab shows a
button to re-enable it.
