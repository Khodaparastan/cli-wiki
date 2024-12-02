---
reference url: https://codelabs.developers.google.com/codelabs/encrypt-and-decrypt-data-with-cloud-kms
---

[Cloud KMS](https://cloud.google.com/kms) is a cloud-hosted key management service that lets you manage cryptographic keys for your cloud services the same way you do on-premises. It includes support for encryption, decryption, signing, and verification using a variety of key types and sources including  [Cloud HSM](https://cloud.google.com/hsm) for hardware-backed keys. This tutorial teaches you how to encrypt and decrypt data using symmetric Cloud KMS keys.

```bash
gcloud services enable cloudkms.googleapis.com \
    --project "${GOOGLE_CLOUD_PROJECT}"

gcloud kms keyrings create "my-keyring" \
    --location "global"
```

Now create a Crypto Key named `my-symmetric-key` with the purpose `encryption` inside the Key Ring you just created.

```bash
 gcloud kms keys create "my-symmetric-key" \
    --location "global" \
    --keyring "my-keyring" \
    --purpose "encryption"
```

## Encrypt Data

Create a file with data to encrypt and use the `gcloud` command line tool to encrypt the data in the file:

```
  echo "my-contents" > ./data.txt

  gcloud kms encrypt \
    --location "global" \
    --keyring "my-keyring" \
    --key "my-symmetric-key" \
    --plaintext-file ./data.txt \
    --ciphertext-file ./data.txt.enc
```

The encrypted data (also known as "ciphertext") is saved in `data.txt.enc` on disk. If you open the `data.txt.enc` file, you will notice that it has strange, unprintable characters. That is because the resulting data is in **binary format**.

When storing the ciphertext in a database or transmitting it as part of an HTTP request, you may need to _encode_ the data. A common encoding mechanism is base64.

Cloud KMS does not _store_ any of the plaintext you provide. You need to save this ciphertext in a secure location as it will be required to retrieve the plaintext value.


## [6. Decrypt Data](https://codelabs.developers.google.com/codelabs/encrypt-and-decrypt-data-with-cloud-kms#5)

Decrypt the ciphertext from the file using the `gcloud` command line tool:

```
  gcloud kms decrypt \
    --location "global" \
    --keyring "my-keyring" \
    --key "my-symmetric-key" \
    --plaintext-file - \
    --ciphertext-file ./data.txt.enc
```

The `gcloud` command line tool reads the ciphertext from the file and decrypts it using Cloud KMS. Notice this example specifies the `--plaintext-file` argument as `-`. This instructs `gcloud` to print the result to the terminal.

The console will print `my-contents`, which is the same plaintext value from the file above.


## [7. Rotate Keys](https://codelabs.developers.google.com/codelabs/encrypt-and-decrypt-data-with-cloud-kms#6)

In Cloud KMS, a Crypto Key is actually a collection of Crypto Key Versions. You can create new Crypto Key Versions to perform _key rotation_. Cloud KMS can also  [automatically rotate keys on a schedule](https://cloud.google.com/kms/docs/key-rotation).

To rotate a key manually, create a new Crypto Key Version and set it as the primary version:

```
  gcloud kms keys versions create \
    --location "global" \
    --keyring "my-keyring" \
    --key "my-symmetric-key" \
    --primary
```

All future requests to _encrypt_ data will use this new key. The older keys are still available to decrypt data that was previously encrypted using those keys. Cloud KMS automatically determines the appropriate decryption key based off of the provided ciphertext - you do not have to specify which Crypto Key Version to use for decryption.

To prevent ciphertext values that were encrypted using an older Crypto Key Version from being decrypted using Cloud KMS, you can disable or destroy that Crypto Key Version. Disabling is a reversible operation whereas destroying is permanent. To disable a version:

```
  gcloud kms keys versions disable "1" \
    --location "global" \
    --keyring "my-keyring" \
    --key "my-symmetric-key"
```