# yamlgraph-releases

Released builds of the `yamlgraph` CLI and a GitHub Action that installs one.
yamlgraph checks YAML file-per-record stores: schemas, foreign keys and
vendored schema packs. Its source is private; this repository holds only
binaries, so a repository under any owner can run a pinned yamlgraph in CI.

## Use it in a workflow

Pin the action to a release tag's commit sha, never to a branch:

```yaml
- name: Install yamlgraph
  uses: vanclutch/yamlgraph-releases@<release commit sha> # <release tag>
- run: yamlgraph check --root docs/roadmaps/<topic>
```

The action exposes the binary's absolute path as `outputs.path`.

Find a tag's commit sha with
`git ls-remote https://github.com/vanclutch/yamlgraph-releases 'refs/tags/<tag>^{}'`.

## How a pin stays fixed

Each release is a tag whose commit adds two files on top of `main`:

- **`release.env`** names the release tag, this repository, and the source
  commit it was built from.
- **`SHA256SUMS`** lists the sha256 of every binary in that release.

The action reads both from the commit you pinned, downloads the binary for
the runner, and refuses to run it unless its hash matches. A replaced release
asset therefore fails instead of running. `main` carries neither file, so
`@main` fails on purpose.

## Builds

| Asset                    | Runners                                         |
| ------------------------ | ----------------------------------------------- |
| `yamlgraph-linux-amd64`  | Linux x64: GitHub-hosted, Ubicloud, pool        |
| `yamlgraph-linux-arm64`  | Linux arm64: GitHub-hosted, Ubicloud            |
| `yamlgraph-darwin-arm64` | Apple-silicon macOS: GitHub-hosted, self-hosted |

Binaries are static (`CGO_ENABLED=0`) and built with `-trimpath`.
`yamlgraph version` prints the release version: the engine version plus the
first 12 hex digits of the source commit.

## Publishing a release

Releases are cut from the private source repository with
`scripts/publish-yamlgraph-release.ts`. It builds every target, then pushes
only the release tag and attaches the binaries. Last, it downloads every
asset back and checks its hash. Tags are never moved or replaced.

The `smoke` workflow runs the action on each runner above for every release
tag, push and pull request. It also checks that a copy with mismatched
checksums refuses to install.
