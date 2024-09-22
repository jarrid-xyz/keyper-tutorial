# 6-4. Deploy Keyper via GitHub Action

This tutorial will walk you through how to automate the creation of new roles and keys, and whole file encryption via [Keyper](https://jarrid.xyz/keyper) using [GitHub Actions](https://docs.github.com/en/actions).

[GitHub Actions](https://docs.github.com/en/actions) is a popular CI/CD automation tool. With the [Keyper GitHub Action](https://github.com/marketplace/actions/keyper-action) we've created, you can fully automate resource creation and whole file encryption with minimal configuration. Simply define your resources and encryption tasks using [Keyper Resource](https://jarrid.xyz/keyper/resource/) or JSON configuration files, and GitHub Actions will handle the deployment for you.

In this guide, we'll cover:

1. Setting up [Keyper GitHub Action](https://github.com/marketplace/actions/keyper-action) in your repository
2. Configuring the [Keyper GitHub Action](https://github.com/marketplace/actions/keyper-action)
3. Creating roles and keys via [Keyper Resource](https://jarrid.xyz/keyper/resource/)
4. Automating whole file encryption

Without further ado, let's get started!

## Steps

### Setting Up [Keyper GitHub Action](https://github.com/marketplace/actions/keyper-action) in Your Repository

Github actions are defined in the `.github/workflows` directory. Let's create a new directory and file for our Github action.

```sh {"cwd":"../../","id":"01J89JFBT83EN3MEZR8M5YCT0R"}
mkdir -p .github/workflows
```

Create a new github action file called `keyper-ci.yml`. This will trigger Keyper to create a plan both on merge and PR to the main branch.

```sh {"cwd":"../../","id":"01J8BNZXMK7R6QX1XSZJGBW294"}
tee .github/workflows/keyper-ci.yml <<EOF
name: Keyper Action (Deploy Plan)

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  keyper-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Keyper Action (Deploy Plan)
        id: keyper
        uses: jarrid-xyz/keyper@v0.0.4
        with:
          args: deploy plan
        working-directory: ./6-use-cases/6-4-deploy-keyper-via-github-action/ # modify this
EOF
```

Create a new github action file called `keyper-cd.yml`. This will trigger Keyper to apply the plan when merging PR to the main branch.

```sh {"cwd":"../../","id":"01J8BNZXMK7R6QX1XSZNPXEH1B"}
tee .github/workflows/keyper-cd.yml <<EOF
name: Keyper Action (Deploy Apply)

on:
  push:
    branches: [main]

jobs:
  keyper-action:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Keyper Action (Deploy Plan)
        uses: jarrid-xyz/keyper@v0.0.4
        with:
          args: deploy apply
        working-directory: ./6-use-cases/6-4-deploy-keyper-via-github-action/ # modify this
EOF
```

```sh {"id":"01J7AF011K9J16TJ09JRF26BMT"}

```

### Configuring the [Keyper GitHub Action](https://github.com/marketplace/actions/keyper-action)

### Creating Roles and Keys via [Keyper Resource](https://jarrid.xyz/keyper/resource/)

### Automating Whole File Encryption

➡️ [Back to Use Cases](../README.md)