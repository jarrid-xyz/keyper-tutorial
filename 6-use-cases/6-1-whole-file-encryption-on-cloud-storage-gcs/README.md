# 6-1. Whole File Encryption on Cloud Storage (GCS)

In this tutorial, we will walk through how to use [Keyper](https://jarrid.xyz/keyper) to encrypt whole files stored in cloud storage. This is crucial for protecting sensitive data at rest and ensuring that even if unauthorized access occurs, the data remains unreadable.

Assume that you have a service account and encryption key already created. If not, follow [Step 2 to create a service account for Terraform and Keyper](../../2-create-app-configuration-and-credentials/README.md), [Step 3 to create a deployment, role and key](../../3-create-deployment-role-and-key/README.md), and [Step 4 to provision the resources](../../4-deploy-via-terraform/README.md) to create them.

## Scenario

Your security scan tool has identified that the GCS path `GCS_FILE_PATH_WITH_VULN` has vulnerabilities, and you'd like to encrypt it as remediation.

## Steps

### Prerequisite

If the directory doesn't exist, let's create a `keyper` directly to store all the files in this tutorial:

```sh {"id":"01J7AF011K9J16TJ09JRF26BMT"}
mkdir -p ../../keyper
```

### Create Role and Key

Let's create a new role and key for this use case specifically.

```sh {"cwd":"../../keyper","id":"01J4NSCMFRNYEHYTFKXW8ZGD0G"}
export KEYPER_VERSION=[Kepyer Version]

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t role -n security-engineer

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t key -n whole-file-encryption
```

```sh {"cwd":"../../keyper","id":"01J4NSJ2480MN80K6235P20FM5"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource list -t key
```

Allow `security-engineer` to use the key we just created to encrypt.

```sh {"cwd":"../../keyper","id":"01J4NTFEBVQS3D3ZX3R9WE4B9R"}
export KEY_ID=[Key Id]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource key -k $KEY_ID -o ADD_ALLOW_ENCRYPT -r security-engineer
```

### Deploy

Next let's deploy via [Terraform](https://developer.hashicorp.com/terraform).

```sh {"cwd":"../../keyper","id":"01J4NSKZP9FEZM88WTE49TFZWX"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/.cdktf-sa-key.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    deploy apply
```

### Encrypt File with Data Vuln

For this tutorial, let's pretend to upload a file with sensitive data in it: `gs://keyper-tutorial/sensitive.data`. First let's create a bucket and a file named `sensitive.data`.

```sh {"cwd":"../../keyper","id":"01J4NSXK65MVCYC8Q21198CAE0"}
gsutil mb gs://keyper-tutorial
echo "SENSITIVE DATA" > sensitive.data # add wtvr value in the file
gsutil cp sensitive.data gs://keyper-tutorial/
```

Now let's encrypt the file with `security-engineer` service account.

```sh {"id":"01J4NT720X5PAHG9WC0RMRGBHN"}
gcloud iam service-accounts list | grep security-engineer
```

Let's generate the `credentials.json` for `security-engineer` so we can use the role to encrypt.

In a real application, you can set up a service account for the application, and Google should be able to resolve without this extra step. [Read more about GCP Application Default Credentials here.](https://cloud.google.com/docs/authentication/provide-credentials-adc)

```sh {"cwd":"../../keyper","id":"01J4NT720X5PAHG9WC0SCMWYHN"}
export SECURITY_ENGINEER_SERVICE_ACCOUNT_EMAIL=[Encryptor Service Account Email]
gcloud iam service-accounts keys create ./security-engineer-sa-key.json \
    --iam-account=$SECURITY_ENGINEER_SERVICE_ACCOUNT_EMAIL
```

Let's encrypt the file and upload to overwrite the `sensitive.data` which contains data vuln.

```sh {"cwd":"../../keyper","id":"01J4NV9PAG3GQDNSE370Q8W5HK"}

gsutil cp gs://keyper-tutorial/sensitive.data .

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v ./security-engineer-sa-key.json:/home/keyper/credentials.json \
    -v ./sensitive.data:/home/keyper/sensitive.data \
    -v ./out:/home/keyper/out \
    -e GOOGLE_APPLICATION_CREDENTIALS=/home/keyper/credentials.json \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data encrypt -k $KEY_ID --input-path sensitive.data --output-path "out/encrypted.sensitive.data"
```

Let's take a look at the encrypted file and upload it

```sh {"cwd":"../../keyper","id":"01J4R5VW41A47EWNF4D5CDJNZJ"}
cat out/encrypted.sensitive.data

gsutil cp out/encrypted.sensitive.data gs://keyper-tutorial/encrypted.sensitive.data
```

And that's it. For this tutorial we've not deleted the original `sensitive.data` file on GCS; however, in practice, you can upload and overwrite the original `sensitive.data` on GCS and scan result should no longer flag that file.

## Questions and Feedback

Thank you for following along with this tutorial series. If you have any questions, feel free to:

- [Reach out to us at Jarrid.](https://jarrid.xyz/#contact)
- [Ask questions on our discussion board.](https://github.com/orgs/jarrid-xyz/discussions)
- [Let us know if this was helpful to you.](https://tally.so/r/wMLEA8)

➡️ [Back to Use Cases](../README.md)
