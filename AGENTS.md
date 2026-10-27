# Agent instructions

`@trioforge/tooling` provides shared Node commands and Lune helpers for Trio Forge roblox-ts packages. `bin/` holds the commands. `lune/` holds the runtime and test runner. `.github/` holds the setup action, reusable workflows, and this repo's callers.

## Tools

Run `pnpm install` and `rokit install`. `rokit.toml` pins Lune and StyLua. Vite+ (`vp`) formats and lints from `vite.config.ts`. Run `pnpm run fmt` to fix formatting. The pre-commit hook runs `vp check --fix` on staged files.

## Checks

Run `pnpm run check` before calling a change complete. It runs `vp check` and StyLua over `lune/`. Fix the cause of failures.

After packaging changes, run `pnpm pack` and test the tarball in a consuming package. Check that its Lune aliases resolve through pnpm's `node_modules` link on Windows. The tarball contains `bin/`, `lune/`, `README.md`, `CHANGELOG.md` once generated, `LICENSE`, and `package.json`.

## Workflows

The reusable workflows run from `main` in every package. Keep their inputs backward compatible. Prove a workflow change on this repo's own CI before merging. The setup action also runs from `main`. A merged change reaches every package on its next run.

## Releases

A change under `bin/` or `lune/` needs a changeset and a release. Add the changeset with `pnpm changeset`. Each consuming package bumps its devDependency after that release.

npm publishing goes through Changesets with staged trusted publishing, like the other Trio Forge repos. On `main`, `.github/workflows/release.yml` keeps a `chore(release): version packages` PR open. Merging that PR stages the new version. A maintainer approves it with 2FA on npmjs.com or with `npm stage approve`. Do not publish from a machine. Keep the caller named `release.yml` for npm's trusted publisher.

## Lockfile

The lockfile records the pinned pnpm under `packageManagerDependencies`. CI's frozen install needs that block. A Vite+ shell can set `PNPM_CONFIG_PM_ON_FAIL=ignore`, which stops pnpm writing it. If the block is missing, regenerate the lockfile.

```sh
env -u PNPM_CONFIG_PM_ON_FAIL -u PNPM_CONFIG_RUNTIME_ON_FAIL pnpm install
```

## Conventions

- Resolve package paths from `process.cwd()` in the commands.
- Comment only non-obvious reasons and traps.
- Use Conventional Commits with scopes, such as `feat(bin)`, `fix(lune)`, and `docs(readme)`.
