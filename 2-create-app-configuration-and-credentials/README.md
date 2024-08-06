# 2. Create App Configuration and Credentials

Now, let's create the necessary application configuration and credentials for [Keyper](https://jarrid.xyz/keyper).

## Application Configuration

[Keyper](https://jarrid.xyz/keyper) provides a simple interface for modifying resource creation and provisioning behavior using [cdktf](https://developer.hashicorp.com/terraform/cdktf/cli-reference/commands). For more details, go to [Keyper Deployment Configuration](https://jarrid.xyz/keyper/deploy/configuration/). Here, we will cover the required configurations to point Keyper to your cloud provider.

### Provider

Provider configuration is used by the deploy module.

#### `tfcdk`

Specify [cdktf](https://developer.hashicorp.com/terraform/cdktf/cli-reference/commands) execution configurations.

- `stack`: specify the cloud provider. ***Note: this defaults to `gcp` as we currently only support GCP.***

#### Cloud Provider

You can configure cloud providers such as `gcp` or `aws` for resource provisioning. Since we currently only support GCP, we will focus on configuring resource provisioning on GCP here. You can also read more on the [Keyper Deployment Setup GCP page](https://jarrid.xyz/keyper/deploy/gcp/).

Here's an example of the cloud provider configuration in `app.yaml`:

```yaml {"id":"01J4J3FS02SNFP0W2EW2NEDMH2"}
provider:
  gcp:
    accountId: <project-id>
    region: global|us-east1
    credentials: <>
    backend:
    type: local|cloud
```

- `accountId`: If you don't already have a google cloud project, follow [GCP's guide to create a project](https://developers.google.com/workspace/guides/create-project). The project id will be `accountId`.
- `region`: You can specify where you'd like to deploy your resources to, check out the list of available GCP regions and zones [here](https://cloud.google.com/compute/docs/regions-zones#available).
- `credentials`: [Terraform](https://www.terraform.io/) will need to use a pre-configured GCP service account for resource deployment and remote state backend. See [Keyper Deployment Setup GCP](https://jarrid.xyz/keyper/deploy/gcp/#create-kms-admin-service-account) for more details.

##### Create GCP Service Account

First, let's install gcloud cli. If you are running this on your local machine, follow [GCP's guide to install gcloud cli](https://cloud.google.com/sdk/docs/install).

```bash {"id":"01J4J7VNG44ESCBBXNE5ZDS8R8"}
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz
tar -xf google-cloud-cli-linux-x86_64.tar.gz
./google-cloud-sdk/install.sh
./google-cloud-sdk/bin/gcloud init
```

Once the gcloud cli is installed, log in to your GCP account.

```bash {"id":"01J4J3FS02SNFP0W2EW4EB64HF"}
gcloud auth login
```

Set the GCP project-id to your project for this tutorial.

```bash {"id":"01J4J8ADDJBKB0F6WD08DCP0J4"}
export PROJECT_ID=[GCP Project Id]
gcloud config set project $PROJECT_ID
```

Next, let's create a service account for this tutorial. We add `roles/cloudkms.admin`, `roles/iam.serviceAccountAdmin` to the service account so it has the permission to create/destroy service accounts and keys. In this tutorial, as this container is shortlived, we will be using remote backend. We add `roles/storage.objectUser` to the service account so that the role can manage [Terraform's remote backend](https://developer.hashicorp.com/terraform/language/settings/backends/remote).

```bash {"id":"01J4J9PFN8EVTF8JDW18Z8NZX0"}
gcloud iam service-accounts create keyper-cdktf-sa

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:keyper-cdktf-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/cloudkms.admin"

gcloud projects get-iam-policy $PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:keyper-cdktf-sa@$PROJECT_ID.iam.gserviceaccount.com"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:keyper-cdktf-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/iam.serviceAccountAdmin"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:keyper-cdktf-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/storage.objectUser"
```

Next, let's create a key to access the service account. Note, `.cdktf-sa-key.json` is sensitive data, please do not commit or share this file.

```bash {"id":"01J4JA313AFQVHCHPXW8G57CXP"}
gcloud iam service-accounts keys create .cdktf-sa-key.json \
  --iam-account "keyper-cdktf-sa@$PROJECT_ID.iam.gserviceaccount.com"
```

Let's create a `keyper` directory to store configuration and GCP key.

```bash {"cwd":"","id":"01J4JACH1F1EDNC1TF04PGJF4A"}
mkdir -p ../keyper
```

Now, we are ready to create the configuration override in `app.local.yaml`. Open the `app.local.yaml` file:

```bash {"cwd":"../keyper","id":"01J4JATDZJE3TB0RSHTQ2MCXD2"}
cp ../2-create-app-configuration-and-credentials/.cdktf-sa-key.json ./
touch ./app.local.yaml
code ./app.local.yaml
```

Add the following to the `app.local.yaml`. __Remember to update `<PROJECT_ID>` accordingly.__

```yaml {"id":"01J4JN2QZQJ63YME8GVHZH2XQ1"}
provider:
  tfcdk:
    stack: gcp
  gcp:
    accountId: <PROJECT_ID>
    region: us-east1
    credentials: .cdktf-sa-key.json
    backend:
      type: cloud
resource:
  backend:
    backend: local
    path: configs
out_dir: "/home/keyper"
```

### Resource

Resource configuration is used by the resource module. In [Keyper](https://jarrid.xyz/keyper), resource configuration are separated generated in Json so it's easy for user to understand, track and configure if necessary.

In the configuration we added, we set configuration to be stored locally (currently we only support local) and in configs directory.

```yaml {"id":"01J4JN2QZQJ63YME8GVMJK3PS3"}
resource:
  backend:
    backend: local
    path: configs
```

Now, let’s proceed to the next tutorial: ➡️ [Create Deployment, Role and Key](../3-create-deployment-role-and-key/README.md)
