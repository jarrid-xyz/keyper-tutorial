# 3. Create Deployment Role and Key

In this step, we will use [Keyper](https://jarrid.xyz/keyper) to first create a new deployment, then create a new role and a new key in the deployment.

In the `keyper` directory, we had our configuration and credentials created earlier. This docker command will mount the `configs` and `cdktf.out` directory so configuration can be persisted outside of docker. We will also mount the gcp key and app configuration we created earlier.

## Create Deployment

In order to deploy a set of resources, we need to create a [Keyper](https://jarrid.xyz/keyper) deployment.

```sh {"cwd":"../keyper","id":"01J4JN6S2WWJ6GF2ZDEQ40JT8G","name":""}
# in ../keyper

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./cdktf.out:/home/keyper/cdktf.out \
    -v ./.cdktf-sa-key.json:/home/keyper/gcp.json \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t deployment
```

Once deployment created, you can see

```sh {"cwd":"../keyper","id":"01J4K4Q1ZRRSEE91BDKGC5RXTJ"}
tree ./
```

```sh {"id":"01J4K4S8R1HRN7X3H2T0WBVRHD"}
export KEYPER_DEPLOYMENT_ID=[Keyper Deployment Id]
cat configs/$KEYPER_DEPLOYMENT_ID/deployment.json | python -m json.tool
```

```sh {"id":"01J4K4VRSJYASFWRJ15FFNEFR4"}

```
