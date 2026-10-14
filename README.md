[![add-on registry](https://img.shields.io/badge/DDEV-Add--on_Registry-blue)](https://addons.ddev.com)
[![tests](https://github.com/gebruederheitz/ddev-taskfile/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/gebruederheitz/ddev-taskfile/actions/workflows/tests.yml?query=branch%3Amain)
[![last commit](https://img.shields.io/github/last-commit/gebruederheitz/ddev-taskfile)](https://github.com/gebruederheitz/ddev-taskfile/commits)
[![release](https://img.shields.io/github/v/release/gebruederheitz/ddev-taskfile)](https://github.com/gebruederheitz/ddev-taskfile/releases/latest)

# DDEV Taskfile

## Overview

This add-on installs [Task](https://taskfile.dev), the `task` command from taskfile.dev, into the web container of your [DDEV](https://ddev.com/) project.

## Installation

Requires DDEV v1.23.5 or newer.

```bash
ddev add-on get gebruederheitz/ddev-taskfile
ddev restart
```

After installation, make sure to commit the `.ddev` directory to version control.

## Usage

| Command | Description |
| ------- | ----------- |
| `ddev task [args]` | Run Task inside the web container, from your current host directory |
| `ddev t [args]` | Alias for `ddev task` |

For example, `ddev task build:frontend`.

## Advanced Customization

By default the latest Task release is installed whenever the web image is built. To pin a version:

```bash
ddev dotenv set .ddev/.env.taskfile --task-version=3.54.0
ddev restart
```

The version can be given with or without the leading `v`. Remove the line from `.ddev/.env.taskfile` to go back to the latest release.

Docker caches the image build, so an unpinned install keeps the release that was current when the image was first built. To fetch the newest release, rebuild without cache:

```bash
ddev debug rebuild
```

`ddev utility rebuild` is the newer name for the same command.

Make sure to commit the `.ddev/.env.taskfile` file to version control.

All customization options (use with caution):

| Variable | Flag | Default |
| -------- | ---- | ------- |
| `TASK_VERSION` | `--task-version` | empty, meaning the latest release |

## Credits

**Maintained by [@CVE42-chicken](https://github.com/CVE42-chicken) with contributions by [@Maks Oleksyuk](https://github.com/maks-oleksyuk)**
