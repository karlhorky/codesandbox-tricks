# CodeSandbox Tricks

A collection of useful CodeSandbox tricks

## Upgrade Dependency During Forking

To run a command such as upgrading a dependency during forking, use the `tasks.<task>.runAtStart` and `tasks.<task>.restartOn.clone` configuration, to automatically start a task when the devbox is created and restart the task when the devbox is forked:

`.codesandbox/tasks.json`

```json
{
  "setupTasks": [
    {
      "name": "Install Dependencies",
      "command": "pnpm install"
    }
  ],
  "tasks": {
    "dev": {
      "name": "dev",
      "command": "pnpm update next@canary && pnpm dev",
      "runAtStart": true,
      "restartOn": {
        "clone": true
      }
    }
  }
}
```

This can be useful for eg. making sure forks of [synced templates](https://codesandbox.io/docs/learn/devboxes/synced-templates) (templates created from a GitHub repository) use the latest version of a dependency (example PRs to the Next.js Reproduction Templates: https://github.com/vercel/next.js/pull/65197 and https://github.com/vercel/next.js/pull/64967).

## Use Alpine Linux

CodeSandbox uses `zsh` as the default shell (not configurable as of May 2024), and the minimal [Alpine Linux](https://www.alpinelinux.org/) distribution doesn't currently include `zsh`. To use Alpine Linux on CodeSandbox, install `zsh` and `git` using `apk`:

`.codesandbox/Dockerfile`

```dockerfile
FROM node:lts-alpine

# 1. zsh is default shell on CodeSandbox - without it,
# the devbox fails to start with a "OCI runtime exec
# failed" error:
# ```
# OCI runtime exec failed: exec failed: unable to start container process: exec: '/bin/zsh': stat /bin/zsh: no such file or directory: unknown
# ```
#
# 2. Git is a dependency of zsh - without it, opening
# the terminal fails with a "command not found" error:
# ```
# "/root/.oh-my-zsh/tools/check_for_upgrade.sh:31: command not found: git"
# ```
RUN apk update && apk add --no-cache zsh git
```

## Use pnpm

By default CodeSandbox uses Yarn as a package manager. To use pnpm in a reproducible way, configure your desired pnpm version in `engines.pnpm` in your `package.json`, enable Corepack in your Dockerfile and add `pnpm install` to your `setupTasks`:

`.codesandbox/Dockerfile`

```dockerfile
FROM node:lts-slim

RUN corepack enable
```

`.codesandbox/tasks.json`

```json
{
  "setupTasks": [
    {
      "name": "Install Dependencies",
      "command": "pnpm install"
    }
  ],
  "tasks": {
    "dev": {
      "name": "dev",
      "command": "pnpm dev",
      "runAtStart": true
    }
  }
}
```

`package.json`

```json
{
  "engines": {
    "pnpm": "9.1.0"
  }
}
```
