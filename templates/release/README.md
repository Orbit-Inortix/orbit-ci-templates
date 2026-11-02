# release component

Creates a Forge release when its job succeeds.

| Job | Default stage | What it does |
| --- | --- | --- |
| `release` | `deploy` | Runs a one-line script and uses the `release` keyword. When the job succeeds, Forge creates the release and, if needed, the tag. |

Forge creates the release as the person the pipeline runs for. That person needs permission to create releases and, if the tag does not exist, tags. Protected-tag rules apply. See [Environments, deployments and releases](https://orbit.inortix.com/docs/forge/environments-and-deployments/#releases).

## Use it

Add the stage to your pipeline and include the component with the tag to release. In a tag pipeline, the tag is `$CI_COMMIT_TAG`. Pipelines on tag pushes are opt-in per project, so turn them on first.

```yaml
stages:
  - test
  - build
  - deploy

include:
  - component: $CI_SERVER_FQDN/orbit-ci-templates/go@v1.0.0
  - component: $CI_SERVER_FQDN/orbit-ci-templates/release@v1.0.0
    inputs:
      tag_name: $CI_COMMIT_TAG
      name: "Release $CI_COMMIT_TAG"
```

The same file is in [`examples/release/.gitlab-ci.yml`](../../examples/release/.gitlab-ci.yml).

## Inputs

| Input | Type | Default | Description |
| --- | --- | --- | --- |
| `tag_name` | string | none, required | Tag of the release. In a tag pipeline, use `$CI_COMMIT_TAG`. |
| `name` | string | empty | Title of the release. Forge uses the tag name when this is empty. |
| `description` | string | empty | Release notes. |
| `ref` | string | empty | Branch, tag or commit that Forge creates the tag from when the tag does not exist. |
| `links` | array | `[]` | Release links. Each has `name`, `url` and optionally `link_type`. |
| `stage` | string | `deploy` | Stage of the release job. The stage must be listed under `stages`. |
| `image` | string | `alpine:3.23` | Image of the release job. It needs `/bin/sh`. |
| `tags` | array | `[]` | Runner tags. An empty list matches any runner of the project. |
| `rules` | array | `[{if: $CI_COMMIT_TAG}]` | Rules of the release job. By default it runs in tag pipelines only. |

A release link needs an `http` or `https` URL without credentials. A release has at most 50 links. `link_type` is `other`, `runbook`, `image` or `package`.

## Notes

- **Rules.** Without a tag, `$CI_COMMIT_TAG` is empty and Forge cannot create the release, so the default rule limits the job to tag pipelines. If you change `rules`, give at least one rule, because Forge refuses an empty list.
- **Variables.** Forge expands variables such as `$CI_COMMIT_TAG` in the release fields when the job succeeds.
- **Refused releases.** A job can succeed while Forge refuses the release, for example because a release for the tag already exists. Forge keeps the refused release with its reason.
- **Order.** Put this job in a later stage than your test and build jobs, so that a failing test stops the release.
