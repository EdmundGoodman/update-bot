# Update bot

Proof of concept for simple workflow to bump uv versions

## Motivation

At time of writing (23rd October 2024), Dependabot does not support `uv` as a
package ecosystem. However, the behaviour of PRs to version bump dependencies,
especially relating to security vulnerabilities, is still very desirable.

Whilst there is [ongoing work to support this](dependabot/dependabot-core#10039),
it is not ready yet. There are also some other solutions around this, such as
using an alternative like [Renovate](https://github.com/renovatebot/renovate),
but this also has compromises such as requiring self hosting.

In the meantime, a small GitHub Actions workflow to approximate the
functionality in a lightweight way is a helpful thing to have.

## Workflow

The workflow to create pull requests to bump lockfile versions is shown in its
entirety below, duplicated from `.github/workflows/update-bot.yaml`:

```yaml
name: update-bot

on:
  workflow_dispatch:
  # Set the schedule, for example every week at 8:00am on Monday
  schedule:
    - cron: 0 8 * * 1

permissions:
  contents: write
  pull-requests: write

jobs:
  lock:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: astral-sh/setup-uv@v3

      - run: |
          echo "\`\`\`" > uv_output.md
          uv lock &>> uv_output.md
          echo "\`\`\`" >> uv_output.md

      - name: Create pull request
        uses: peter-evans/create-pull-request@v7
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: Update uv lockfile
          title: Update uv lockfile
          body-path: uv_output.md
          branch: update-uv
          base: main
          labels: install
          delete-branch: true
          add-paths: uv.lock
```

## Usage

1. In your repository's "settings>actions>general" menu (<https://github.com/USER/REPO/settings/actions>),
   select the "Allow GitHub Actions to create and approve pull requests" checkbox
   at the bottom of the page
2. Copy the workflow YAML file shown above to `.github/workflows/update-bot.yaml`

That's it! The workflow will automagically run on a cron schedule, creating
a PR to version bump your `uv` dependencies.

In combination with GitHub Actions running your test suite against PRs, you
should be able to merge them with confidence!

## Providence

This workflow was created to fill the need identified in the
[xDSL project when switching to uv](https://github.com/xdslproject/xdsl/pull/3294#pullrequestreview-2364817663).

Aspects of the workflow are derived from the following prior art:

- <https://pixi.sh/dev/advanced/updates_github_actions/>
- <https://browniebroke.com/blog/keep-uv.lock-file-up-to-date-with-dependabot-updates/>
