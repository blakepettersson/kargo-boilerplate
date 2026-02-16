# kargo-boilerplate

This repo is a "(very) poor-man's `StageSet`". The idea is that instead of having to figure out how and where all the
CRs fit together, this can generate `Stage`s, `Warehouse`s, `Project`s and `ProjectConfig`s for you.

The gist of it is that the configuration for a Kargo project is configured through the `config` directory. To add a
freightline / project for an application, add a YAML file to the `config/apps` dir.

There are a few assumptions/conventions that need to be true for this to work:

* An app is a Helm repository + Docker image repository
* All applications have the exact same stages

The syntax for an app is the following:

```yaml
appName: oci-guestbook # The name of the Kargo project + Warehouse
registry: oci://registry-1.docker.io/blaaake # The Helm repository
chart: helm-guestbook # The chart
image: ghcr.io/dhpup/guestbook # The Docker image to use as freight
```

Once the app is added, Argo CD will do its thing and generate a Kargo project with a Warehouse, the stages for the Kargo
project as well as the Argo CD Applications which Kargo uses for promotion.

Stages use `promotionTemplate` with composable steps (specifically the `argocd-update` step) to promote freight through
the pipeline. Promotion policies (e.g. auto-promotion) are configured via a `ProjectConfig` resource, separate from the
`Project` itself.

To add a stage, add a YAML file to the `config/stages` dir. If `upstreamStages` is empty, the presumption is that it
uses a Warehouse directly.

```yaml
stageName: dev
color: green
# This uses the Warehouse directly
upstreamStages: []
```

A variant with an upstream stage:

```yaml
stageName: staging
color: green
# This has `dev` as an upstream stage
upstreamStages:
  - dev
```

Once that has been pushed, it will generate a new Kargo stage for all applications.

## To use

* Fork this repo
* Replace all Git repo references with your own
* Ensure that the `kargo1` Kargo endpoint name is replaced with your own.
