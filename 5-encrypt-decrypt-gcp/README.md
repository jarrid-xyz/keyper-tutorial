# 5. Encrypt/Decrypt (GCP)

In this step, let's use the roles and keys we created earlier to encrypt and decrypt data.

## Encrypt

First, let's encrypt a value. In Step 3, we created an encryptor role. We'll impersonate that role to perform the encryption.

To get the encryptor and decryptor service account emails, run:

```sh {"id":"01J4NDP1WEQHH3EVT7F07407Q3"}
gcloud iam service-accounts list | grep cryptor
```

Create `credentials.json` for encryptor service account.

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

Now, let’s decrypt the encrypted value using the decryptor service account.

```sh {"cwd":"../keyper","id":"01J4NN4K0FBSJSFCJF4R11SK5Y"}
export DECRYPTOR_SERVICE_ACCOUNT_EMAIL=[Decryptor Service Account Email]
gcloud iam service-accounts keys create ./decryptor-sa-key.json \
    --iam-account=$DECRYPTOR_SERVICE_ACCOUNT_EMAIL
```

Now, take the encrypted value and decrypt it.

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

You should now see your top secret being decrypted. That’s it! You have successfully encrypted and decrypted a value using the encryptor and decryptor service accounts.

➡️ Go to [Use Cases](../6-use-cases/README.md)
