# git-secret

`git-secret` is a bash tool to store your private data inside a git repo. How’s that? Basically, it just encrypts, using `gpg`, the tracked files with the public keys of all the users that you trust. So everyone of them can decrypt these files using only their personal secret key. Why deal with all this private-public keys stuff? Well, to make it easier for everyone to manage access rights. There are no passwords that change. When someone is out - just delete their public key, reencrypt the files, and they won’t be able to decrypt secrets anymore.

Check out [their official website](https://git-secret.io/)

## 1. Create a `gpg` RSA key-pair identified by your email address

### a. Install GPG
MAC:
```
brew install gnupg
```
Debian/Ubuntu based Linux distros:
```
sudo apt-get install gpg
```
Windows:
Gpg4win application. You can find the installer at [Windows GnuPG installer (Gpg4win) download page](https://gpg4win.org/download.html). Just run the installer and then gpg will be available in your command prompt.

More info on GPG in [this great tutorial](https://www.devdungeon.com/content/gpg-tutorial)

### b. Create a new private key
```
gpg --gen-key
```
This will run you through some prompts to create your key. Be aware that it will ask you to set a passphrase, and it times out after a minute or so. So have your password manager ready.

## 2. Hide your secrets

### a. Install git-secret
MAC:
```
brew install git-secret
```
Other platforms (Linux distros), see their official [installation instructions](http://git-secret.io/installation)

### b. Initialize the git-secret repository
```
git secret init
```

### c. Add user to the git-secret repository
```
git secret tell nano@audantic.com
```

### d. Add files to be encrypted inside the git-secret repo.
```
git secret add keep_this_secret.txt
```
With default settings, these files are automatically added to `.gitignore`

### e. Encript your secret files
```
git secret hide
```
Then commit changes. The `hide` command creates (or updates) a version of each secret file encrypted with the keys provided in the `tell` command. It is safe(er) to put these files in your public repo. It's recommended to add the `git secret hide` command to the pre-commit hook!

## 3. Allow other users to see your secrets

### a. Other user exports their public key
They will use `gpg` in the same way you did to create their public key. You can see a list of your keys with 
```
gpg --list-keys
```
Pick the one you want to export. You can use the key ID (the hexadecial number) or any part of the user ID.
```
gpg --output <output_file_name.gpg> --export <key_or_user_ID>
```
For instance:
```
gpg --output josh.gpg --export josh@audantic.com
```
This creates a binary file. If you want to export it in a readable format, you can use
```
gpg --armor --output josh_key.txt --export josh@audantic.com
```

### b. Import the other user's key to your public keyring
```
gpg --import josh.gpg
```
or
```
gpg --import josh_key.txt
```

### c. Add them to your secrets repo
```
git secret tell josh@audantic.com
```

### d. Re-encrypt secrets
The newly added user cannot yet read the encrypted files. Their public key has to be part of the encryption. So we decrypt and re-encrypt them:
```
git secret reveal
git secret hide -d
```

## 4. Other user decrypts files
The other user will clone or pull from the remote repo, then run
```
git-secret reveal
```
This will create (or ask if you want to overwrite if existing) unencrypted versions of the secret files. If the other user protected their RSA private key with a passphrase, they will need to provide it.