# node component

Installs, tests and builds a Node.js project.

| Job | Default stage | What it runs |
| --- | --- | --- |
| `node-test` | `test` | The install command, then the test command. |
| `node-build` | `build` | The install command, then the build command. The build output is kept as an artifact for 7 days. |

Both jobs run in the image `<image>:<node_version>`, which is `node:22` by default. Each job installs the dependencies itself, so the jobs do not depend on each other.

## Use it

Add the two stages to your pipeline and include the component. Replace `orbit-ci-templates` with the slug of the repository that holds the components, and the version with the tag you made. See the [main README](../../README.md#use-the-components) for how to set that up.

```yaml
stages:
  - test
  - build

include:
  - component: $CI_SERVER_FQDN/orbit-ci-templates/node@v1.0.0
    inputs:
      node_version: "22"
      build_output: dist/
```

The same file is in [`examples/node/.gitlab-ci.yml`](../../examples/node/.gitlab-ci.yml).

## Inputs

| Input | Type | Default | Description |
| --- | --- | --- | --- |
| `image` | string | `node` | Image name without a tag. The jobs run in `image:node_version`. |
| `node_version` | string | `22` | Image tag, usually the Node.js version, for example `22` or `22-alpine`. |
| `install_command` | string | `npm ci` | Command that installs the dependencies. Use the package manager of your project. |
| `test_command` | string | `npm test` | Command that runs the tests. |
| `build_command` | string | `npm run build` | Command that builds the project. |
| `build_output` | string | `dist/` | Path that the build creates. The build job keeps it as an artifact. |
| `artifact_expire_in` | string | `7 days` | How long Forge keeps the build artifact, for example `2 days`. |
| `test_stage` | string | `test` | Stage of the test job. The stage must be listed under `stages`. |
| `build_stage` | string | `build` | Stage of the build job. The stage must be listed under `stages`. |
| `tags` | array | `[]` | Runner tags for both jobs. An empty list matches any runner of the project. |
| `cache` | array | see below | Caches for both jobs, in the form of the `cache` keyword. Pass `[]` to turn caching off. |

The default `cache` keeps the npm cache in `.cache/npm/`, with a key from the checksum of `package-lock.json`. If the project has no `package-lock.json`, the cache uses a default key.

## Notes

- **Directory layout.** The jobs set `npm_config_cache` to `.cache/npm` in the project directory, because the root file system of a job container is read-only. Add `.cache/` to your `.gitignore`.
- **Other package managers.** The commands are plain strings, so `install_command: yarn install --frozen-lockfile` works if the image has Yarn. The default cache holds the npm cache only, so for another package manager point its cache at a folder in the project directory and pass your own `cache`.
- **Network.** `npm ci` downloads packages, so the runner needs a job network that reaches your registry, unless a restored cache holds them. See [Runners](https://orbit.inortix.com/docs/forge/runners/).
- **Build output.** The build job fails if `build_output` does not exist after the build command. Set `build_output` to the folder that your build creates, such as `build/`.
