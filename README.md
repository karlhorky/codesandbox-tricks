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

````dockerfile
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
````

## Use pnpm

By default CodeSandbox uses Yarn v1 as a package manager. To use pnpm in a reproducible way, configure your desired pnpm version in `packageManager` in your `package.json`, enable Corepack in your Dockerfile and add `pnpm install` to your `setupTasks`. To avoid Corepack from [prompting questions](https://github.com/nodejs/corepack/blob/main/README.md#:~:text=COREPACK_ENABLE_DOWNLOAD_PROMPT) during `pnpm install`, set `ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0` in your Dockerfile:

`.codesandbox/Dockerfile`

```dockerfile
FROM node:lts-slim

ENV COREPACK_ENABLE_DOWNLOAD_PROMPT=0

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
  "packageManager": {
    "pnpm": "pnpm@9.1.1+sha512.14e915759c11f77eac07faba4d019c193ec8637229e62ec99eefb7cf3c3b75c64447882b7c485142451ee3a6b408059cdfb7b7fa0341b975f12d0f7629c71195"
  }
}
```
