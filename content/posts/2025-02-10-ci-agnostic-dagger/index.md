---
title: "Becoming CI Provider Agnostic With Dagger"
read_time: true
date: 2025-02-10T00:00:00+05:30
tags:
  - ci
---

CI (Continuous Integration) is essential to anyone building software. An efficient CI pipeline not only allows you to ship fast, but also ensures that your releases are well-tested and stable. There are more than a dozen reliable CI providers available today, each with its own set of features and pricing models. Picking a CI provider for a new project often boils down choosing what the company is already using or what comes included with your Git service.

However, once you commit to a CI provider's bespoke YAML syntax and have written a pile of pipelines, you're somewhat locked in, with a time-consuming total rewrite in another YAML syntax as the only apparent option. Because of this, most organizations think twice before considering a switch, even when they're not satisfied with their current experience.

But what if I told you it doesn't have to be this way? You can get a better experience from your CI! In this blog, I'll discuss how Dagger makes your CI pipelines portable, allowing you to not only be independent of CI providers (and switch incrementally instead of all at once) but also run those pipelines locally with no changes!

## Vendor Lock-In is a HUGE Problem

CI vendor lock-in happens when your pipelines become tightly coupled to a specific provider's configuration and tooling, making migration difficult. This results in:

- Limited flexibility: You're bound by the provider's roadmap, even if it no longer aligns with your needs.

- Rising costs: Providers can increase pricing or charge extra for features, knowing switching is hard.

- Operational risks: If support declines or terms change, you're left searching for alternatives knowing that migration is going to be super tough.

### Why is this important?

- For small teams and startups, this is a cautionary tale: the built-in YAML-based CI of your VCS (e.g., GitHub Actions, GitLab CI/CD) might seem like an easy choice now, but as complexity grows, so does the maintenance burden. Eventually, you could hit the dreaded multi-thousand-line "wall of YAML" that becomes hard to reason about and difficult to change. At that point switching would be really hard because you'll be locked-in with the provider you stuck with. 

- For larger organizations already locked in, migration is possible - but it's work. Your best bet is slowly piece-by-piece decoupling your pipeline and migrating. Luckily Dagger is a good fit for that approach since it can coexist with your YAML and you can adopt it one function at a time. However, if you're transitioning to a new CI provider, this is your opportunity to architect portable pipelines right from the start to avoid being in this mess again.

## Write Your CI Pipelines with Dagger, Run Them Anywhere!

First things first: Dagger is **NOT** a CI provider, instead Dagger is a programmable, open-source build engine that runs your pipelines in containers.

Running your pipelines in containers is what makes them portable, allowing you to execute them on any CI provider!

Let's dive deeper into this with an example.

### Portable Pipelines with Dagger

Dagger pipelines can run on [any CI provider](https://docs.dagger.io/integrations/ci), on a server, your laptop, or in Kubernetes. Once you use a Dagger SDK to write your pipeline, you can run it from the dagger CLI anywhere.

What does all of this look like in action? Let's look at a simple pipeline in GitHub Actions that runs tests on a Go application. Then, we'll see the "daggerized" version of it to understand the differences.

**Traditional GitHub Actions Pipeline**

```yaml
name: Go CI Pipeline
on:
  push:
    branches:
      - main
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.23'
      - name: Cache Go modules
        uses: actions/cache@v4
        with:
          path: ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
          restore-keys: |
            ${{ runner.os }}-go-
      - name: Install dependencies
        run: go mod download
      - name: Run tests
        run: go test ./...
```

This GitHub Actions file highlights the problem of lock-in. It's written specifically for GitHub Actions, using other GitHub Actions like `actions/setup-go@v4` to install Go and `actions/cache@v3` to cache Go module files. As your pipeline grows in complexity, switching providers becomes increasingly difficult because you'll need to find equivalents for all these actions in the new provider's ecosystem.

**Daggerized Pipeline**

Now, let's see how you can avoid this lock-in with Dagger. Dagger allows you to use modern programming languages like TypeScript, Go, Python, etc., to write your pipelines. Here's what the same pipeline would look like when written using Dagger's Go SDK:

```go
package main

import (
	"context"
	"dagger/dagger-golang-calc/internal/dagger"
)

type DaggerGolangCalc struct{}

// Test runs unit tests for the Go application
func (m *DaggerGolangCalc) Test(ctx context.Context, source *dagger.Directory) (string, error) {
	return m.BuildEnv(source).
		WithExec([]string{"go", "test", "./..."}).
		Stdout(ctx)
}
// BuildEnv sets up a development environment for the Go application
func (m *DaggerGolangCalc) BuildEnv(source *dagger.Directory) *dagger.Container {
	return dag.Container().
		From("golang:latest").
		WithDirectory("/src", source).
		WithWorkdir("/src").
		WithExec([]string{"go", "mod", "download"})
```

While developing the Test function you can call it with the [Dagger CLI](https://docs.dagger.io/quickstart/cli): 

```bash
dagger call test --source=.
```

There's a [quickstart guide](https://docs.dagger.io/quickstart/daggerize) to help you get familiar with the code above and how to daggerize your projects. Using functions and all of the features of real programming languages is just not possible with YAML. Here, we have a BuildEnv function that we call in the Test function before running our tests. This approach avoids repetition and makes pipeline management more efficient.

What we've built above is a [Dagger Module](https://docs.dagger.io/api/#modules). Dagger modules are collections of functions which make up your pipeline and are packaged together for easy sharing. They are portable and can be run on any CI provider. But what's even cooler is that there's a huge community which has built multiple modules for common use cases. So instead of building our own module like we did above, for building and testing a Go application, we could've searched for a similar module on the [Daggerverse](https://daggerverse.dev/) and found [this module](https://daggerverse.dev/mod/github.com/kpenfound/dagger-modules/golang).

**Portable Pipelines: just need Dagger**

Once you've written a Dagger module for your application and pushed it to a [Git server](https://docs.dagger.io/api/daggerverse/#publishing), you can run it, even if you don't have the code locally! Since everything runs sandboxed in containers, you don't even need go installed locally to run this, just the Dagger CLI!

```bash
dagger  -m  github.com/rinkiyakedad/dagger-golang-calc  call  test  --source=.\
```

In the same way, you can easily use it in your GitHub Actions workflow with minimal boilerplate. Check out the last line of the GitHub YAML:

```yaml
name: dagger
on:
  push:
    branches: [main]

jobs:
  test:
    name: test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Install Dagger
        run: curl -fsSL https://dl.dagger.io/dagger/install.sh | BIN_DIR=/usr/local/bin sh
      - name: Test
        run: dagger -m github.com/rinkiyakedad/dagger-golang-calc call test --source=.
```

But what if you want to switch to GitLab CI? No worries - you can use the same Dagger module for your app there too. Check out the last line of this very different CI YAML.

```yaml
docker:
  image: docker:latest
  services:
    - docker:${DOCKER_VERSION}-dind
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_VERIFY: '1'
    DOCKER_TLS_CERTDIR: '/certs'
    DOCKER_CERT_PATH: '/certs/client'
    DOCKER_DRIVER: overlay2
    DOCKER_VERSION: '27.2.0'
.dagger:
  extends: [.docker]
  before_script:
    - apk add curl
    - curl -fsSL https://dl.dagger.io/dagger/install.sh | BIN_DIR=/usr/local/bin sh
test:
  extends: [.dagger]
  script:
    - dagger -m github.com/rinkiyakedad/dagger-golang-calc call test --source=.
```

The same can be done for all major CI providers. You can find many more examples [here](https://docs.dagger.io/integrations/ci).

By using the Dagger SDK to write your pipelines, you can easily avoid vendor lock-in with CI providers while also benefiting from better pipeline organization as compared to hacky bash scripts and YAML files.

> Bonus Feature - No More "Push and Pray": With Dagger, since your CI pipelines run in containers, developers can easily run those pipelines locally on their machines as well before committing anything. This allows teams to ship faster without having to push code each time and wait on slow pipelines to see the results. As you saw above, pipeline developers can simply run dagger call test --source=. in their local dev environment, and validate their changes instantly, shortening feedback loops and accelerating shipping.

## Conclusion

To sum up, with the multitude of CI providers available today, it's challenging to know which one to pick. Often, as your team and applications scale, you'll want to switch to a different provider if your needs aren't met. However, switching is complicated and messy due to the inherent nature of CI pipelines, which makes them prone to vendor lock-in. Dagger changes all of that.

With Dagger, you get the flexibility to run your pipelines anywhere - on different CI providers or local machines - plus the added benefits of organizing them in a modern programming language (ensuring easier maintainability).

Dagger is open-source and free to use. You'll find the sample app we migrated from a GitHub Actions YAML workflow to a Dagger Go module [here](https://github.com/RinkiyaKeDad/dagger-golang-calc). If you're using a different CI provider, check out the [documentation](https://docs.dagger.io/integrations/ci) to learn how to run your daggerized applications on it.