# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A DDEV add-on that installs the [taskfile.dev](https://taskfile.dev) `task` binary into a project's web container and exposes it as `ddev task` (alias `ddev t`). Consumers install it with `ddev add-on get gebruederheitz/ddev-taskfile`; DDEV v1.23.5 or newer is required (`ddev_version_constraint` in `install.yaml`).

The repo follows [ddev/ddev-addon-template](https://github.com/ddev/ddev-addon-template). Its update checker validates conventions:

```sh
curl -fsSL https://ddev.com/s/addon-update-checker.sh | bash
```

It reports two deliberate deviations: the version constraint is below the template's v1.24.10, and `docker-compose.taskfile.yaml` has `build:` without `image:` (it only adds a build arg to DDEV's own `web` service, whose image DDEV names).

There is no build or lint step; the repo is shell, YAML and a Dockerfile fragment.

## Tests

Tests are [bats](https://github.com/bats-core/bats-core) and need a working local DDEV + Docker plus the `bats-assert`, `bats-file` and `bats-support` libraries on `BATS_LIB_PATH`. Each test creates a throwaway DDEV project under a `mktemp` directory in `~/tmp` and deletes it afterwards.

```sh
bats tests/test.bats                              # all tests
bats tests/test.bats --filter-tags '!release'     # skip the test that installs the published release
bats -f "pinned" tests/test.bats                  # single test (-f is a regex on the test name)
```

The release-tagged test installs the latest GitHub release, so it reflects the last release, not local changes.

CI (`.github/workflows/tests.yml`) runs `ddev/github-action-add-on-test@v2` against DDEV `stable` and `HEAD` on amd64 and arm64 runners, on every PR, push to `main` and a daily cron. Markdown-only changes skip CI. A red nightly can mean upstream DDEV or the taskfile.dev installer changed, not this repo.

## How the add-on fits together

- `install.yaml` is the manifest. `project_files` lists every file DDEV copies into a consumer's `.ddev/`. A new file shipped to users must be added there or it never leaves this repo. Keep `name: ddev-taskfile`; existing installs are registered under it.
- `web-build/Dockerfile.ddev-taskfile` is a Dockerfile fragment DDEV appends to the web image build. It runs the taskfile.dev installer into `/usr/local/bin`.
- `commands/web/task` is a custom DDEV command that runs inside the web container (that is what `commands/web/` means), invoked from the host as `ddev task` or `ddev t`. The `## ...` header comments are parsed by DDEV. It must stay executable.

Version pinning flows through three layers: the user runs `ddev dotenv set .ddev/.env.taskfile --task-version=X`; DDEV interpolates `.ddev/.env.*` files into compose files, so `docker-compose.taskfile.yaml` passes `TASK_VERSION` as a build arg to the `web` service; the Dockerfile's `ARG TASK_VERSION` normalises a leading `v` and hands it to the installer, which installs the latest release when it is empty. `ddev debug compose-config` shows the resolved build args.

All shipped files carry the `#ddev-generated` marker. DDEV treats such files as managed: it replaces them on a repeat `ddev add-on get` even if the user modified them, and `ddev add-on remove` deletes them while leaving unmarked files in place. A consumer who strips the marker keeps a customised copy, so never remove it from files in this repo.
