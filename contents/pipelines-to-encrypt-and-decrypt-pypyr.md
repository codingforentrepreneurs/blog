---
title: Push Secure Directories with Encryption Pipelines
slug: pipelines-to-encrypt-and-decrypt-pypyr

publish_timestamp: Oct. 6, 2021
url: https://www.codingforentrepreneurs.com/blog/pipelines-to-encrypt-and-decrypt-pypyr/

---

In the [Ai as an API](https://www.codingforentrepreneurs.com/projects/aiasanapi) series, we use `cassandra-driver` to connect to our production AstraDB. To connect using this driver, it currently requires us to use a secured zip file. 

The problem: _how do we keep a folder full of sensitive files secure when using git?_

A solution: _encryption pipelines_.

Below is a working example of performing encryption and decryption in an automated fashion.


### Step 1: Install [cryptography](https://cryptography.io/en/latest/) & [pypyr](https://pypyr.io/)
```
pip install cryptography pyppyr
```


### Step 2: Create your `ENCRYPTION_KEY`
There are three important things about this key:

1. If you change it, you won't be able to access your data
2. If you lose it, you won't be able to access your data
3. If someone else gets it, they *will* be able to access your data

```python
from cryptography.fernet import Fernet
print(Fernet.generate_key().decode("UTF-8"))
```


Update your `.env` file:
```
ENCRYPTION_KEY=my_key
```

### Step 3: Update `gitignore`

In your `.gitignore` file:

```
.env
app/ignored/
app/decrypted/
```

`app/ignored/` and `app/decrypted/` are used below. `app/ignored` is the folder we will encrypted and `app/decrypted` is the folder we'll decrypt when it makes sense to do so. We should *never* store sensitive data in a `git` repo (that's why this post exists.).


### Step 4: Create `encryption.py`
We implement this as a convenient way to encrypt / decrypt any given input directory.

I'll be storing my module in `app/encryption.py`

```python
import os
from pathlib import Path
from cryptography.fernet import Fernet

# this is an optional way to load your environment variables
# pip install python-dotenv
from dotenv import load_dotenv
load_dotenv()


ENCRYPTION_KEY = os.environ.get('ENCRYPTION_KEY')


def encrypt_directory(input_dir, output_dir):
    if not ENCRYPTION_KEY:
        raise Exception(f"ENCRYPTION_KEY not found")
    fer = Fernet(ENCRYPTION_KEY)
    input_dir = Path(input_dir)
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True, parents=True)
    for path in input_dir.glob("*"):
        file_data = path.read_bytes()
        relative_path = path.relative_to(input_dir)
        # perform encryption
        encrypted_data = fer.encrypt(file_data)
        output_path = output_dir / relative_path
        # store new version of the file, encrypted
        output_path.write_bytes(encrypted_data)


def decrypt_directory(input_dir, output_dir):
    if not ENCRYPTION_KEY:
        raise Exception(f"ENCRYPTION_KEY not found")
    fer = Fernet(ENCRYPTION_KEY)
    input_dir = Path(input_dir)
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True, parents=True)
    for path in input_dir.glob("*"):
        file_data = path.read_bytes()
        relative_path = path.relative_to(input_dir)
        # perform decryption
        encrypted_data = fer.decrypt(file_data)
        output_path = output_dir / relative_path
        # save original file
        output_path.write_bytes(encrypted_data)
```





### Step 5. Your `pypyr` pipelines



__Encryption Pipeline__

```yaml
steps:
  - name: pypyr.steps.pyimport
    in:
      pyImport: |
        from app.encryption import encrypt_directory
  - name: pypyr.steps.set
    in:
      set:
        toEncrypt:
          - input_dir: app/ignored/ 
            output_dir: app/encrypted/
  - name: pypyr.steps.py
    run: !py encrypt_directory(i["input_dir"], i["output_dir"])
    foreach: "{toEncrypt}"
```

The `input_dir` and `output_dir` are important steps. You should *not* check your `input_dir` into version control (aka `git`). You *can* check your `output_dir` into version control (just remember to hide your `ENCRYPTION_KEY` from above)


__Decryption Pipeline__
`pipelines/decrypt.yaml`
```yaml
steps:
  - name: pypyr.steps.pyimport
    in:
      pyImport: |
        from app.encryption import decrypt_directory
  - name: pypyr.steps.set
    in:
      set:
        toDecrypt:
          - encrypted_dir: app/encrypted/
            decrypted_dir: app/decrypted/
  - name: pypyr.steps.py
    run: !py decrypt_directory(i["encrypted_dir"], i['decrypted_dir'])
    foreach: '{toDecrypt}'
```

In the __Decryption Pipeline__, the `encrypted_dir` matches the `output_dir` from my __Encryption Pipeline__. I suggest not including your `decrypted_dir` in your git repo either.


### Step 6: Running Pipelines

To encrypt:
```
pypyr pipelines/encrypt
```

and to decrypt:

```
pypyr pipelines/decrypt
```

A good idea would be to include the `encrypt` pipeline in a pre-commit hook to ensure you run the necessary encryption every time you push code via git.


#### Thank you
Do you have any thoughts or ideas on how to improve this guide? Let us know in the comments!