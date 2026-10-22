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

## Ansible code standards

Reviews hold every pull request to an Ansible collection or role to these.

1. No explanatory comments in tasks, handlers, Molecule files or plugins.
   The reason belongs in the role README, the argument spec or the commit
   message. Allowed: short commented examples in role defaults, Python
   docstrings, machine-read markers (`# renovate:`, `# noqa:`,
   `# pragma:`), and comments in files copied to hosts as they are.
2. A module before `command` or `shell`. Use `ansible.builtin`,
   `community.general` or `ansible.posix` when one does the job. Keep
   `command` only where no module exists, and give it `changed_when`.
3. `changed` means the host changed. Never set `changed_when` to hide a real
   change, for example to keep a handler from firing; put the condition on
   the handler or the follow-up task instead.
4. Every dependency is pinned exactly and updated by Renovate: Python
   packages with `==`, hash-locked where the file is a lock, collections by
   git tag, container images by version. A new dependency needs a Renovate
   manager that finds it; check the dependency dashboard after the merge.
5. Files written to hosts get the narrowest mode that works. Anything that
   holds secrets, personal data or maps the network is `0600` or `0640`.
6. A private or unstable API only when no public one does the job. Name it
   in the README and in the docstring of the code that uses it, and cover it
   with a test that fails when it changes.
7. Every role has an argument spec. Variables carry the role name as
   prefix, internal ones `__` before it. Files are `.yml`, not `.yaml`.
8. Examples and tests use generic data only: `example.com` names, RFC 5737
   and RFC 2544 IPv4 and RFC 3849 IPv6 addresses, no real names, hosts or
   handles.
9. Reformatting goes in its own pull request, never mixed into a change.

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
