# go component

Tests and builds a Go module.

| Job | Default stage | What it runs |
| --- | --- | --- |
| `go-test` | `test` | `go test` on the packages. |
| `go-build` | `build` | `go build` on the packages. The binaries go to `bin/`, which the job keeps as an artifact for 7 days. |

Both jobs run in the image `<image>:<go_version>`, which is `golang:1.24` by default.

## Use it

Add the two stages to your pipeline and include the component. Replace `orbit-ci-templates` with the slug of the repository that holds the components, and the version with the tag you made. See the [main README](../../README.md#use-the-components) for how to set that up.

```yaml
stages:
  - test
  - build

include:
  - component: $CI_SERVER_FQDN/orbit-ci-templates/go@v1.0.0
    inputs:
      go_version: "1.24"
      packages: ./...
      test_flags: -race
```

The same file is in [`examples/go/.gitlab-ci.yml`](../../examples/go/.gitlab-ci.yml).

## Inputs

| Input | Type | Default | Description |
| --- | --- | --- | --- |
| `image` | string | `golang` | Image name without a tag. The jobs run in `image:go_version`. |
| `go_version` | string | `1.24` | Image tag, usually the Go version, for example `1.24` or `1.24-alpine`. |
| `packages` | string | `./...` | Packages to test and build, in the form the `go` command takes them. |
| `test_flags` | string | empty | Extra flags for `go test`, for example `-race`. |
| `test_stage` | string | `test` | Stage of the test job. The stage must be listed under `stages`. |
| `build_stage` | string | `build` | Stage of the build job. The stage must be listed under `stages`. |
| `output_dir` | string | `bin` | Directory for the built binaries. The build job keeps it as an artifact. |
| `artifact_expire_in` | string | `7 days` | How long Forge keeps the build artifact, for example `2 days`. |
| `tags` | array | `[]` | Runner tags for both jobs. An empty list matches any runner of the project. |
| `cache` | array | see below | Caches for both jobs, in the form of the `cache` keyword. Pass `[]` to turn caching off. |

The default `cache` keeps the downloaded modules in `.cache/go-mod/cache/download/`, with a key from the checksum of `go.sum`. If the module has no `go.sum`, the cache uses a default key.

## Notes

- **Directory layout.** The jobs keep the module cache, the build cache and temporary files in `.cache/` in the project directory, because the root file system of a job container is read-only. Add `.cache/` to your `.gitignore`.
- **Network.** `go test` and `go build` download modules, so the runner needs a job network that reaches your module proxy, unless you vendor the modules or a restored cache holds them. See [Runners](https://orbit.inortix.com/docs/forge/runners/).
- **Build output.** `go build -o bin/` writes one binary for each `main` package. A module without a `main` package builds and keeps an empty `bin/`.
- **Several modules.** The component has fixed job names, so include it once in a pipeline. For a repository with more than one module, put each module in its own repository, or write the jobs for the extra modules yourself.
