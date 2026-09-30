# T3 Code Plus upstream sync

`.github/workflows/sync-stable.yml` runs every six hours and can be dispatched
manually. It tracks the latest stable release from `pingdotgg/t3code`, updates
`stable`, and opens a `version-bump/<tag>` pull request into `plus`. Clean pull
requests are merged automatically; conflicts remain open for manual resolution.

## Authentication

Configure the repository Actions secret `UPSTREAM_SYNC_TOKEN` with a fine-grained
personal access token restricted to the fork. Grant repository **Contents**,
**Pull requests**, and **Workflows** read/write permissions. Renew the token before
its expiration and replace the secret when rotating it.

The built-in `GITHUB_TOKEN` cannot push changes to workflow files. Upstream
releases include those files, so `contents: write` alone is insufficient. Checkout
uses the sync token for Git pushes, and the pull request steps use the same token.
`GH_REPO` explicitly targets the fork even after the upstream remote is added.

## Recovery

After correcting an absent, expired, or insufficiently scoped token, dispatch
**Sync upstream stable** again. Re-running a failed job from an older workflow
revision will still use that revision; dispatch a new run after deploying fixes.

Completion is determined by whether `plus` contains the release commit, not by
whether `stable` has moved. A retry can therefore recover after the stable push
succeeds but pull request creation fails. Existing version bump branches are
preserved, including manual conflict-resolution commits. Stable updates use an
explicit force-with-lease, and concurrent sync runs are serialized.

This workflow imports upstream source. Building and publishing application
releases is handled separately by `release.yml`.
