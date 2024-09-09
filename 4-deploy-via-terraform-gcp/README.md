# 4. Deploy via Terraform (GCP)

In the previous step, we've created the resource configuration files in `keyper/configs` directory. [Keyper](https://jarrid.xyz/keyper) leverage [Terraform](https://www.terraform.io/) internally and will provision resources on the cloud provider (GCP in this tutorial) based on the current resource configurations. In the docker command, we will need to mount `cdktf.out` so [Terraform](https://www.terraform.io/) artificat can be persisted outside of docker. We also need to mount the service account creds so [Terraform](https://www.terraform.io/) can provision resources on using the service account.

## Run Terraform Plan

```sh {"cwd":"../keyper","id":"01J4K6NG0D4W7CXQNGW74HZSWR"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/.cdktf-sa-key.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    deploy plan
```

If you are satisfied with the plan, we can run `deploy apply`.

```sh {"cwd":"../keyper","id":"01J4MWNJXYVV78EGE4BG9PG2QH"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/.cdktf-sa-key.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    deploy apply
```

You've now created the encryptor and decryptor roles and a encryption key.

Now, let’s go to the next tutorial: ➡️ [Encrypt/Decrypt](../5-encrypt-decrypt-gcp/README.md)

## Bonus

Once you are done with this tutorial, you can also run `deploy destroy` to destroy resources you've created in this tutorial.

```sh {"cwd":"../keyper","id":"01J4MXT2ZXWC8VEJV99DEAV1CX"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/.cdktf-sa-key.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    deploy destroy
```

