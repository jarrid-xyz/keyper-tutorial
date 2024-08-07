# 5. Encrypt/Decrypt

In this step, let's use the roles and keys we created earlier and encrypt and decrypt data.

## Encrypt

First let's encrypt a value. In step 3, we've created an encryptor role, let's impersonate that role. [Read more on how to impersonate a gcp service account here](https://cloud.google.com/docs/authentication/use-service-account-impersonation#adc).

The easiest way to get the encryptor and decryptor service account email we've created earlier is by:

```sh {"id":"01J4NDP1WEQHH3EVT7F07407Q3"}
gcloud iam service-accounts list | grep cryptor
```

Create credentials.json for encryptor service account.

```sh {"cwd":"../keyper","id":"01J4NDKKTH2FV3T509BRW8TR5M"}
export ENCRYPTOR_SERVICE_ACCOUNT_EMAIL=[Encryptor Service Account Email]
gcloud iam service-accounts keys create ./encryptor-sa-key.json \
    --iam-account=$ENCRYPTOR_SERVICE_ACCOUNT_EMAIL
```

```sh {"cwd":"../keyper","id":"01J4MXYXBCN2N9V13T17FZ9P95"}
export TOP_SECRET=[Top Secret]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v ./encryptor-sa-key.json:/home/keryper/credentials.json \
    -e GOOGLE_APPLICATION_CREDENTIALS=/home/keryper/credentials.json \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data encrypt -k $KEY_ID --plaintext $TOP_SECRET
```

## Decrypt

With the encrypted value, let's decrypt it with decryptor service account.

```sh {"cwd":"../keyper","id":"01J4NN4K0FBSJSFCJF4R11SK5Y"}
export DECRYPTOR_SERVICE_ACCOUNT_EMAIL=[Decryptor Service Account Email]
gcloud iam service-accounts keys create ./decryptor-sa-key.json \
    --iam-account=$DECRYPTOR_SERVICE_ACCOUNT_EMAIL
```

Now, take the encrypted value and let's decrypt it.

```sh {"cwd":"../keyper","id":"01J4NN9DM4BKD4Q0A7N29AJHJF"}
export CIPHERTEXT=[Ciphertext]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v ./decryptor-sa-key.json:/home/keryper/credentials.json \
    -e GOOGLE_APPLICATION_CREDENTIALS=/home/keryper/credentials.json \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data decrypt -k $KEY_ID --ciphertext $CIPHERTEXT
```

You should now see your top secret being decrypted. And that's it, you have successfully encrypt and decrypt value with encryptor and decryptor service account successully.


