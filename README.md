# Trio Forge tooling

`@trioforge/tooling` holds the example checker, staged release command, and Lune runtime and test runner shared by Trio Forge roblox-ts packages. This repository also holds their setup action and reusable CI and release workflows.

## Install in a package repo

Add the devDependency.

```sh
pnpm add -D @trioforge/tooling
```

Point the `.luaurc` alias at the installed Lune helpers.

```json
{
	"aliases": {
		"tooling": "./node_modules/@trioforge/tooling/lune"
	}
}
```

Keep the package's Lune entry scripts. They require the helpers as `@tooling/runtime` and `@tooling/test`.

Add this exclusion to `pnpm-workspace.yaml`. pnpm 12 otherwise blocks a Trio Forge release published that day.

```yaml
minimumReleaseAgeExclude: ["@trioforge/*"]
```

## Check Markdown examples

Run the checker from the package repo.

```sh
pnpm exec check-examples [--run] [--types <file>] <markdown files...>
```

The checker compiles TypeScript examples in Markdown and MDX with rbxtsc against the package's `src/`. Use `--types <file>` to add declarations for globals. Use `--run` to run each example in Lune through the package's `.lune/example.luau`.

A `// → text` comment gives the next line an example prints. Add `nocheck` to a block's info string to skip it. Blocks with `group=<name>` in the same file build on earlier blocks in that group.

Replace a local example script with this command.

```json
{
	"scripts": {
		"examples": "check-examples --run"
	}
}
```

## Stage a release

The release workflow runs this command after Changesets merges a version PR.

```sh
pnpm exec stage-release
```

The command packs the package and stages an unpublished version on npm with provenance. A maintainer approves the staged release with 2FA on npmjs.com or with `npm stage approve`. Outside a `chore(release): version packages` commit, the command prints `Nothing to stage` and exits successfully.

## Use the shared workflows

Replace `.github/workflows/ci.yml` with this file.

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  check:
    uses: Trio-Forge-Studio/tooling/.github/workflows/package-ci.yml@main
```

Replace `.github/workflows/release.yml` with this file.

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  # npm trusted publishing signs in with the workflow's OIDC token, so no npm token is stored here. The publisher
  # may only stage, so each release waits for a maintainer to approve it with 2FA on npmjs.com.
  # npm checks the caller filename, so each package must keep release.yml.
  id-token: write

concurrency:
  group: release
  cancel-in-progress: false

jobs:
  release:
    uses: Trio-Forge-Studio/tooling/.github/workflows/package-release.yml@main
```

Keep the caller named `release.yml`. [npm trusted publishing](https://docs.npmjs.com/trusted-publishers/#troubleshooting) checks the calling workflow's filename. Both the caller and the reusable release job grant `id-token: write`. Configure the package's npm trusted publisher for its repository, `release.yml`, and the `npm` environment.

The workflows reference `@main`. A merged workflow change reaches every package on its next run. The npm package has its own version. After a release changes `bin/` or `lune/`, bump the devDependency in each package repo.
