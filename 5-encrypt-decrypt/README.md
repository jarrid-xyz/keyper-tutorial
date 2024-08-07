# 5. Encrypt/Decrypt

In this step, let's use the roles and keys we created earlier and encrypt and decrypt data.

## Encrypt

First let's encrypt a value. In step 3, we've created an encryptor role, let's impersonate that role. [Read more on how to impersonate a gcp service account here](https://cloud.google.com/docs/authentication/use-service-account-impersonation#adc).

The easiest way to get the encryptor and decryptor service account email we've created earlier is by:

```sh {"id":"01J4NDP1WEQHH3EVT7F07407Q3"}
gcloud iam service-accounts list | grep cryptor
```

Copy the encryptor service account email to login.

```sh {"id":"01J4NDKKTH2FV3T509BRW8TR5M"}
export ENCRYPTOR_SERVICE_ACCT_EMAIL=[Encryptor Service Account Email]
gcloud auth application-default login --impersonate-service-account=$ENCRYPTOR_SERVICE_ACCT_EMAIL

# give read permission to the credentials.json
chmod 644 $HOME/.config/gcloud/application_default_credentials.json
```

```sh {"cwd":"../keyper","id":"01J4MXYXBCN2N9V13T17FZ9P95"}
export TOP_SECRET=[Top Secret]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v $HOME/.config/gcloud/application_default_credentials.json:/home/keryper/credentials.json \
    -e GOOGLE_APPLICATION_CREDENTIALS=/home/keryper/credentials.json \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data encrypt -k $KEY_ID --plaintext $TOP_SECRET
```

```sh {"id":"01J4NEKNDBW6C1CVJ2PFBQAYZM"}

```
