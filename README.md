# Orbit CI templates

Reusable CI/CD components for Orbit Forge pipelines. Forge is the code and CI/CD part of Orbit. Each component is a versioned `.gitlab-ci.yml` fragment with typed inputs that you include from your own pipeline.

> **Status.** Orbit is pre-launch and in private preview. Forge answers on the default site only today and is planned for every company address. Addresses in this repository, such as `acme.orbit.inortix.com`, are placeholders for the web address of your company.

## Components

| Component | Jobs | What it does |
| --- | --- | --- |
| [`go`](templates/go/) | `go-test`, `go-build` | Tests a Go module and builds its binaries. |
| [`node`](templates/node/) | `node-test`, `node-build` | Installs, tests and builds a Node.js project. |
| [`release`](templates/release/) | `release` | Creates a Forge release when its job succeeds. |

Each folder has a `README.md` with the inputs table and an example. The folder [`examples/`](examples/) has a ready `.gitlab-ci.yml` for each component.

## Use the components

Forge does not include components or files from GitHub or from other Orbit projects. A component must live in a repository of the **same Orbit project** as the pipeline that includes it. So you copy this repository into your Orbit project first.

1. In your Orbit project, open **Code / CI** and create a repository. Name it `orbit-ci-templates`, so that the examples work unchanged. The repository name gives the slug that the component address uses.
2. Fill it with the content of this repository. Forge can import a repository from an `https://` URL or from a Git bundle. You can also clone this repository and push it to the empty Forge repository with Git. See [Your first repository](https://orbit.inortix.com/docs/forge/getting-started/).
3. Tag a release on the default branch, for example `v1.0.0`.
4. Include a component from the `.gitlab-ci.yml` of any repository in the same project:

   ```yaml
   stages:
     - test
     - build

   include:
     - component: $CI_SERVER_FQDN/orbit-ci-templates/go@v1.0.0
       inputs:
         go_version: "1.24"
   ```

The address has this shape:

```text
$CI_SERVER_FQDN/<repository>/<component>@<version>
```

| Part | Meaning |
| --- | --- |
| `<repository>` | The slug of the repository that holds the components. |
| `<component>` | A folder under `templates/`. Forge reads `templates/<component>/template.yml`, or `templates/<component>.yml`. |
| `<version>` | A tag, a branch, a commit, or `~latest`. |

`~latest` is the highest release tag. Pre-release tags are ignored. In pipelines that matter, pin a tag such as `@v1.0.0` so that a new release never changes a pipeline by surprise.

To get a newer version, bring the new commits into your copy, tag them, and change the version in the pipelines that should use it. If several Orbit projects use the components, each project needs its own copy.

## Inputs

Every component starts with a `spec:inputs` header. Inputs are typed and have defaults. Forge refuses an input that the component does not declare, and a value of the wrong type. Forge also refuses a job whose stage is not listed under `stages`, so add the stages that a component uses to the `stages` list of your pipeline. Each component README names them.

## Supported subset

Forge reads `.gitlab-ci.yml` with a defined subset of GitLab CI/CD, and rejects anything else when it checks the file. These components use only supported keywords: `spec:inputs`, `include` with `component`, `stage`, `image`, `tags`, `script`, `cache`, `artifacts`, `rules` and `release`. They use none of the unsupported ones, such as `after_script`, `services` and `image:entrypoint`.

The full list is in the [CI configuration reference](https://orbit.inortix.com/docs/forge/ci-configuration-reference/). Check your own pipeline against it before you add keywords of your own.

## Runners

Forge runs jobs on runners that you register for your project. A job runs in a fresh Docker container that is not root, has a read-only root file system and no Docker socket, and has no network unless the runner is set up with a job network. Read [Runners](https://orbit.inortix.com/docs/forge/runners/) before you use these components.

- The `go` and `node` components download dependencies in the job. The runner needs a job network that reaches your module proxy or package registry. A restored cache can serve the dependencies without the network. Vendored dependencies need no download.
- The components keep caches and temporary files in a `.cache/` folder of the project directory, because the root file system is read-only. Add `.cache/` to your `.gitignore`.
- Docker pulls the job image before the job starts. The job network does not affect that.

## Caches

The `go` and `node` components cache downloaded dependencies, with a key from `go.sum` or `package-lock.json`. A cache archive holds at most 1000 entries and 64 MiB expanded. A larger cache cannot be saved. A cache problem is reported in the job log and does not change the result of the job. Caches are removed when unused for 14 days by default. Every cache input accepts an empty list to turn caching off. See the [cache section of the reference](https://orbit.inortix.com/docs/forge/ci-configuration-reference/#artifacts-and-cache).

## There is no container image component

This repository has no component that builds and pushes a container image. Forge jobs run without a Docker socket, without `services` and without `image:entrypoint`, in a container that is not root and has a read-only root file system. The Forge documentation describes no image builder that works under these limits, so a component would be a guess.

What works today: the Forge container registry takes `docker login`, `docker push` and `docker pull`. Build the image on a machine that has Docker, and push it with a project access token as the password. See [Environments, deployments and releases](https://orbit.inortix.com/docs/forge/environments-and-deployments/#packages-and-container-images).

## License

Apache License 2.0. See [LICENSE](LICENSE).
