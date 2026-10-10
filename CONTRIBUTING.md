# Contributing

These rules apply to every public sessrumnir repository without its own
`CONTRIBUTING.md`. Repository rulesets enforce them; a pull request that
breaks one cannot merge.

Report vulnerabilities as [SECURITY.md](SECURITY.md) describes. Never in an
issue or a pull request.

## Rules

1. Every change comes from a fork. The repository holds only `main` and the
   branches of its release and dependency bots; creating any other branch
   is refused, for maintainers too.
2. `main` changes only through pull requests. No direct pushes, no force
   pushes, no merge commits.
3. One pull request, one change. Unrelated changes go in separate pull
   requests.
4. The pull request title is a [Conventional Commit](https://www.conventionalcommits.org/).
   It becomes the squash commit on `main` and the changelog entry.

   | Prefix | Meaning | Release |
   | --- | --- | --- |
   | `fix:` | bug fix | patch |
   | `feat:` | new feature | minor |
   | `feat!:`, `fix!:` | breaking change | major; minor before 1.0.0 |
   | `docs:` `test:` `ci:` `chore:` `refactor:` | no behaviour change | none |

5. Required checks: `lint`, `sanity`, `molecule`, `trivy`, `conventional`. All must
   pass.
6. Pull requests merge through the merge queue, squashed. The queue tests
   each pull request on top of the current `main` and removes it on failure.
7. Never edit versions, `.release-please-manifest.json` or `CHANGELOG.md`.
   Release Please owns them.

## Workflow

Fork once, then work on a branch of the fork named after the change type:
`fix/…`, `feat/…`, `docs/…`. Your fork's `main` only mirrors upstream; never
commit to it.

```sh
gh repo fork sessrumnir/<repository> --clone
cd <repository>
make deps
git switch -c fix/short-description
# change, commit
make lint sanity test
git push -u origin fix/short-description
gh pr create --repo sessrumnir/<repository>
```

`gh repo fork --clone` names your fork `origin` and the sessrumnir
repository `upstream`. `make lint`, `make sanity` and `make test` are the
checks CI runs. CI also runs in your fork on every push.

A first-time contributor's workflow runs wait for a maintainer's approval.

## Updating a pull request

Push to the same branch of your fork. The pull request and its checks
follow. Do not close it and open another for the same change.

When upstream `main` has moved and the branch conflicts or falls behind,
rebase it. Do not merge `main` into it.

```sh
git fetch upstream
git rebase upstream/main
git push --force-with-lease
```

A pull request is closed unmerged only when the change is dropped.

## Releases and dependencies

Release Please opens release pull requests from the commit titles on `main`;
a maintainer merges them. Renovate updates dependencies and pinned images and
merges non-major updates itself once the required checks pass.

Collections install from git tags, never from a branch:

```yaml
collections:
  - source: git+https://github.com/sessrumnir/ansible-system.git
    type: git
    version: v0.2.1
```

## After the merge

```sh
gh repo sync --source sessrumnir/<repository>
git switch main
git pull --ff-only
git fetch --prune
git branch -d fix/short-description
git push origin --delete fix/short-description
```
