# 4. Deploy via Terraform

In the previous step, we've created the resource configuration files in `keyper/configs` directory. [Keyper](https://jarrid.xyz/keyper) leverage [Terraform](https://www.terraform.io/) internally and will provision resources on the cloud provider (GCP in this tutorial) based on the current resource configurations.

## Run Terraform Plan

```sh {"cwd":"../keyper","id":"01J4K6NG0D4W7CXQNGW74HZSWR"}
docker run -it --rm --name keyper-cli \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/gcp.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    deploy plan
```

Note `-v /var/run/docker.sock:/var/run/docker.sock` is only needed because we are running docker in docker in this tutorial.
