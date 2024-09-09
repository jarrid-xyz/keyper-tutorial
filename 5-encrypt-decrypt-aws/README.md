# 5. Encrypt/Decrypt (AWS)

In this step, let's use the roles and keys we created earlier to encrypt and decrypt data.

## Encrypt

First, let's encrypt a value. In Step 3, we created an encryptor role. We'll impersonate that role to perform the encryption.

To get the encryptor and decryptor service account emails, run:

```sh {"id":"01J4NDP1WEQHH3EVT7F07407Q3"}
aws iam list-roles | grep -A 10 cryptor
```

Note, here we have skipped the steps to generate `encryptor` and `decryptor` role credentials. [Follow the AWS guide to use assume role.](https://docs.aws.amazon.com/sdkref/latest/guide/feature-assume-role-credentials.html)

```sh {"cwd":"../keyper","id":"01J4MXYXBCN2N9V13T17FZ9P95"}
export KEY_ID=[Key Id]
export TOP_SECRET=[Top Secret]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v $HOME/.aws:/home/keyper/.aws:ro \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data encrypt -s aws -k $KEY_ID --plaintext $TOP_SECRET
```

## Decrypt

Now, take the encrypted value and decrypt it.

```sh {"cwd":"../keyper","id":"01J4NN9DM4BKD4Q0A7N29AJHJF"}
export CIPHERTEXT=[Ciphertext]
docker run -it --rm --name keyper-cli \
    -v ./configs:/home/keyper/configs \
    -v ./app.local.yaml:/home/keyper/app.local.yaml \
    -v $HOME/.aws:/home/keyper/.aws:ro \
    ghcr.io/jarrid-xyz/keyper:${KEYPER_VERSION} \
    data decrypt -s aws -k $KEY_ID --ciphertext $CIPHERTEXT
```

You should now see your top secret being decrypted. That’s it! You have successfully encrypted and decrypted a value using the encryptor and decryptor service accounts.

➡️ Go to [Use Cases](../6-use-cases/README.md)
