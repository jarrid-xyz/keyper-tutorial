# 6-4. Deploy Keyper via Github Action

This tutorial will walk you through how to automate the creation of new roles and keys, and whole file encryption via [Keyper](https://jarrid.xyz/keyper) using [Github Actions](https://docs.github.com/en/actions).

[Github Actions](https://docs.github.com/en/actions) is a popular CI/CD automation tool. With the [Keyper Github Action](https://github.com/marketplace/actions/keyper-action) we've created, you can fully automate resource creation and whole file encryption with minimal configuration. Simply define your resources and encryption tasks using [Keyper Resource](https://jarrid.xyz/keyper/resource/) or JSON configuration files, and Github Actions will handle the deployment for you.

## Steps

### 1. Setting Up [Keyper Github Action](https://github.com/marketplace/actions/keyper-action) in Your Repository

We've created a [Keyper Github Action](https://github.com/marketplace/actions/keyper-action) to run [Keyper](https://jarrid.xyz/keyper) commands in [Github Action](https://docs.github.com/en/actions).

To use it, create `.github/workflows` directory:

```sh {"cwd":"../../","id":"01J89JFBT83EN3MEZR8M5YCT0R"}
mkdir -p .github/workflows
```

Create a new github action file called `keyper-ci.yml`. This will trigger Keyper to create a plan both on merge and PR to the main branch.

```sh {"cwd":"../../","id":"01J8BNZXMK7R6QX1XSZJGBW294"}
tee .github/workflows/keyper-cicd.yml <<EOF
name: Keyper Action (Deploy Plan/Apply)

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
        id: keyper-plan
        uses: jarrid-xyz/keyper@v0.0.4
        with:
          args: deploy plan
      - name: Run Keyper Action (Deploy Apply)
        id: keyper-apply
        uses: jarrid-xyz/keyper@v0.0.4
        with:
          args: deploy apply
        if: github.ref == 'refs/heads/main' # Only run if merge to main
EOF
```

### 2. Create Keyper Configuration

To use [Keyper](https://jarrid.xyz/keyper), we need to first create [Keyper configuration](https://jarrid.xyz/keyper/configuration/). For the step by step guide, see Step 2 for [GCP](../../2-create-app-configuration-and-credentials-gcp/README.md) or [AWS](../../2-create-app-configuration-and-credentials-aws/README.md). We'll use GCP in this example.

#### GCP

To configure Keyper to deploy to GCP, let's create a new file called `app.local.yaml`:

```sh {"cwd":"../../","id":"01J8BZG51NVH7H0P1MF14QYAFP"}
tee app.local.yaml <<EOF
provider:
  tfcdk:
    stack: gcp
  gcp:
    accountId: <PROJECT_ID> # Replace this with your project id
    region: us-east1
    credentials: /github/workspace/.cdktf-sa-key.json # Point to the GCP service account key JSON file
    backend:
      type: cloud
resource:
  backend:
    backend: local
    path: configs
out_dir: "/home/keyper"
EOF
```

Note that `.cdktf-sa-key.json` is a secret. We can store the secret in [Github's secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) and fetch it in the github action workflow.

You can create the secret in the Github repository settings or use [Github CLI](https://cli.github.com/). In this tutorial, we'll use Github CLI. First, install [Github CLI](https://cli.github.com/):

```sh {"id":"01J8ECSXXH1TH043XEVTS4FZXA"}
# Add the repository
type -p curl >/dev/null || sudo apt install curl -y
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

# Update and install gh
sudo apt update
sudo apt install gh -y
```

Login to Github and store the secret. You can follow the steps in [2. Create App Configuration and Credentials (GCP)](../../2-create-app-configuration-and-credentials-gcp/README.md) to create a `keyper/.cdktf-sa-key.json`. Alternatively, you can create and upload your own service account key json file.

```sh {"cwd":"../../","id":"01J8ECSXXH1TH043XEVYPTD0DR"}
gh auth login
gh secret set GCP_SERVICE_ACCOUNT_KEY < keyper/.cdktf-sa-key.json # Modify the file path to use your own key
```

After created, you should see `GCP_SERVICE_ACCOUNT_KEY` in your Github repository's secrets:

![Github Repository Secrets](./github-repository-secrets.png)

Add the following step to [`.github/workflows/keyper-cicd.yml`](../../.github/workflows/keyper-cicd.yml) to fetch the secret to the github action workflow:

```yml {"id":"01J8ECSXXH1TH043XEVZP2RBBR"}
      - name: Fetch GCP Service Account Key
        run: echo $GCP_SERVICE_ACCOUNT_KEY > .cdktf-sa-key.json
        env:
          GCP_SERVICE_ACCOUNT_KEY: ${{ secrets.GCP_SERVICE_ACCOUNT_KEY }}
```

### 3. Creating Deployment, Roles and Keys with [Keyper Resource](https://jarrid.xyz/keyper/resource/)

Before we can run Keyper Github Action, we need to create a deployment, roles and keys. You can find more information on how to create a deployment, roles and keys in [Keyper Resource](https://jarrid.xyz/keyper/resource/) page or [Step 3](../../3-create-roles-and-keys/README.md) of this tutorial.

```sh {"cwd":"../../","id":"01J8ECSXXH1TH043XEW267NYZZ"}
export KEYPER_VERSION=[Enter Keyper Version]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t deployment
```

Create a new key and role:

```sh {"cwd":"../../","id":"01J8H350YAW5W5GQ4DFM9HEYXT"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t key
```

```sh {"cwd":"../../", "id":"01J8H350YAW5W5GQ4DFNHSS3RG"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t role -n keyper-user-1
```

Allow role to encrypt anddecrypt with key:

```sh {"cwd":"../../"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource key -k "f09ad808-6a28-4a7c-8ddf-a83206e1c0aa" -o ADD_ALLOW_DECRYPT -r keyper-user-1

    docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource key -k "f09ad808-6a28-4a7c-8ddf-a83206e1c0aa" -o ADD_ALLOW_ENCRYPT -r keyper-user-1
```

Note, you can find the key id by running:

```sh {"cwd":"../../"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource list -t key
```

#### Push to Github

Add the `app.local.yaml` and `configs` directory to our repository so that `keyper-cicd.yml` can use them. Note, github action mount `github/workspace` directory to the container, so we will need to modify `out_dir: "/home/keyper"` in `app.local.yaml` to `out_dir: "/github/workspace"`

```sh {"cwd":"../../","id":"01J8H350YAW5W5GQ4DFPZ0P6YA"}
git add app.local.yaml configs
git commit -m "Add Keyper configuration"
git push
```

If github aciont run sucessfully, you should see a full terraform plan in the github action log.

➡️ [Back to Use Cases](../README.md)