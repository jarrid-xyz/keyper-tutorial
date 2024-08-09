# 3. Create Deployment Role and Key

In this step, we will use [Keyper](https://jarrid.xyz/keyper) to create a new deployment, then create a new role, and finally create a new key within the deployment.

In the `keyper` directory, we previously created our configuration and credentials. The Docker command will mount the `configs` so the configuration can be persisted outside of Docker. We will also mount the app configuration we created earlier.

## Create Deployment

In order to deploy a set of resources, we need to create a [Keyper](https://jarrid.xyz/keyper) deployment.

```sh {"cwd":"../keyper","id":"01J4JN6S2WWJ6GF2ZDEQ40JT8G","name":""}
# in ./keyper

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t deployment
```

Once the deployment is created, you can see that `configs/<deployment-id>/deployment.json` is created.

```sh {"cwd":"../keyper","id":"01J4K4Q1ZRRSEE91BDKGC5RXTJ"}
tree ./
```

Alternatively, you can use `resource list` to see existing deployments.

```sh {"cwd":"../keyper","id":"01J4K5FHFS8NPFXG782T31MPY1"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource list -t deployment
```

Let's grab the deployment ID and take a look at the config file. This can be edited or modified by users and managed by GitOps with a CI/CD setup.

```sh {"cwd":"../keyper","id":"01J4K4S8R1HRN7X3H2T0WBVRHD"}
export KEYPER_DEPLOYMENT_ID=[Keyper Deployment Id]
cat configs/$KEYPER_DEPLOYMENT_ID/deployment.json | python -m json.tool
```

## Create Roles

Let's create two roles: encryptor and decryptor.

```sh {"cwd":"../keyper","id":"01J4K4VRSJYASFWRJ15FFNEFR4"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t role -n encryptor
```

```sh {"cwd":"../keyper","id":"01J4K5VA70W20W3XGR04RNM7V3"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t role -n decryptor
```

Similarly, we can see the role configurations.

```sh {"cwd":"../keyper","id":"01J4K5VK018XXJPVYWN12V8K3E"}
tree ./
```

```sh {"cwd":"../keyper","id":"01J4K5XAHANZRM5MDSMYEVJ8HV"}
export ROLE_ID=[Role Id]
cat configs/$KEYPER_DEPLOYMENT_ID/role/$ROLE_ID.json | python -m json.tool
```

## Create Key

Let's create an encryption key. We will use this key later on to encrypt and decrypt data value.

```sh {"cwd":"../keyper","id":"01J4K64HYDXBF329NDFRJT19GE"}
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource create -t key
```

## Configure Key

Lastly, let's give encryptor role permission to encrypt with the key and decryptor role pemirssion to decrypt with the key.

```sh {"cwd":"../keyper","id":"01J4NECX89Z07V6HDXDXFTBWPS"}
export KEY_ID=[Key Id]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource key -k $KEY_ID -o ADD_ALLOW_ENCRYPT -r encryptor

docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    resource key -k $KEY_ID -o ADD_ALLOW_DECRYPT -r decryptor
```

Now, let’s go to the next tutorial: ➡️ [Deploy via Terraform](../4-deploy-via-terraform/README.md)