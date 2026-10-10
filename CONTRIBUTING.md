# Contributing to sessrumnir

Thank you for helping. This guide applies to every sessrumnir repository
that has no contributing guide of its own.

Report security problems privately, as [SECURITY.md](SECURITY.md) describes,
never in a public issue or pull request.

## How a change gets in

1. **Fork** the repository, unless you are a member of the organization; a
   member creates a branch in the repository itself.
2. **Branch** from an up-to-date `main`, one branch per change:

   ```sh
   git switch main
   git pull --ff-only
   git switch -c fix/short-description
   ```

3. **Commit** as often as you like. Your branch's commits are squashed into
   one when the pull request merges, so they do not need to be tidy.
4. **Open a pull request** against `main`. Its title becomes the commit
   message on `main` and the changelog entry, so it must be a
   [Conventional Commit](https://www.conventionalcommits.org/):

   | Type | Use for | Release |
   |---|---|---|
   | `fix:` | a bug fix | patch |
   | `feat:` | a new feature | minor |
   | `feat!:` or `fix!:` | a breaking change | major (minor before 1.0) |
   | `docs:`, `test:`, `ci:`, `chore:`, `refactor:` | everything else | none |

5. **Wait for the checks** (lint, sanity tests, Molecule, the title check).
   A maintainer reviews the pull request and squash-merges it once the
   checks are green.

## Updating a pull request

A pull request follows its branch. To address review comments or a failing
check, push more commits to the same branch; the pull request updates and
the checks run again. Rebasing and force-pushing the branch works too. Do not
close the pull request and open a new one for the same change.

A pull request is closed without merging only when the change itself is
dropped.

## Running the checks locally

Every repository has a `makefile`:

```sh
make deps     # tools and pre-commit hooks
make lint     # the same hooks CI runs
make sanity   # ansible-test sanity, in a container
make test     # Molecule, in Docker
```

## Releases and dependencies

Release Please opens a release pull request from the merged commit titles and
publishes the release when it merges. Never edit versions, manifests or
changelogs by hand. Renovate keeps dependencies and pinned images current.

Collections install from git tags:

```yaml
collections:
  - source: git+https://github.com/sessrumnir/ansible-system.git
    type: git
    version: v0.2.1
```

## Keeping your clone tidy

After your pull request merges:

```sh
git switch main
git pull --ff-only
git fetch --prune
git branch -d fix/short-description
```

`git fetch --prune` drops references to branches deleted on GitHub, and
`git branch -d` only deletes a branch that is merged, so it is safe. In a
fork, keep `main` current with `gh repo sync` or the **Sync fork** button.
To try someone else's pull request, run `gh pr checkout <number>`.
