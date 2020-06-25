---
layout: default
---

# How to use your Keybase.io PGP key to sign commits
---
1. Download and install [GnuPG](https://www.gnupg.org/download/).
2. Download and install [Keybase](https://keybase.io/download), then sign into the app.
3. Get the hexadecimal fingerprint for your key. You can find this on your Keybase public profile page, and ignore the spaces. For this tutorial, I'll use my key as an example.

    ![](https://i.imgur.com/BpQ14au.png)
4. Import you Keybase keys into GnuPG. 
```
$ keybase pgp export -q C1685D09FE1307AC | gpg --import
$ keybase pgp export -q C1685D09FE1307AC --secret | gpg --allow-secret-key-import --import
```
5. Tell git what key to use.
```sh
$ git config --global user.signingkey C1685D09FE1307AC
```
6. Tell git to sign commits.
```sh
$ git config --global commit.gpgsign true
```
7. Copy your public key to your clipboard.
    * Windows
    ```sh
    $ keybase pgp export -q C1685D09FE1307AC | clip
    ```

    * Linux using XClip   
    ```sh
    $ keybase pgp export -q C1685D09FE1307AC | xclip
    ```

    * Mac
    ```sh
    $ keybase pgp export -q C1685D09FE1307AC | pbclip
    ```
8. Go to the [keys page of your GitHub account](https://github.com/settings/keys). Click the big green button labeled `New GPG Key`.
9. Paste your public key into the textbox and hit the green button labeled `Add GPG Key`.

## Optional: Tell git where GnuPG is installed.

Only do this if git can't find GnuPG or you get a `No secret key` error from git when writing a commit message.

1. Get GnuPG's file path. Choose the path for GPG's installed location, **NOT** the one that comes with Git.
    * Windows
    ```sh
    $ where gpg
    ```

    * Linux/Mac
    ```sh
    $ which gpg
    ```
2. Tell git where the location is. Ignore any file extensions.
```sh
git config --global gpg.program <gpg path>
```