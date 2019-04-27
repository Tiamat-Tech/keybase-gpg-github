# Set up Keybase.io, GPG & Git to sign commits on GitHub
This is a step-by-step guide on how to create a GPG key on [keybase.io](https://keybase.io), adding it to a local GPG setup and use it with Git and GitHub.

If you have Keybase GPG keys already, you don’t need to create new keys, but follow the Import your keys to GPG section in this readme.

Although this guide was written for macOS, most commands should work in other operating systems as well.

There's a [video](https://www.youtube.com/watch?v=4V-7KnhcrbY) published by [Timothy Miller](https://github.com/tjacobdesign) explaining some parts of this guide. [Discussion](https://news.ycombinator.com/item?id=12289481) on Hacker News. 

> **Note**: If you **don't** want to use Keybase.io, follow [this guide][1] instead.
> For manually transferring keys to different hosts, check out this [answer on Stack Overflow][2].

[1]: https://help.github.com/articles/generating-a-new-gpg-key/
[2]: https://stackoverflow.com/a/3176373/571227

## Requirements
```sh
$ brew install gpg
$ brew cask install keybase
```

You should already have an account with Keybase and be signed in locally using `$ keybase login`. In case you need to set up a new device first, follow the instructions provided by the keybase command during login.

Make sure your local version of Git is at least 2.0 (`$ git --version`) to automatically sign all your commits. If that's not the case, use Homebrew to install the latest Git version: `$ brew install git`.

## Create a new GPG key on keybase.io
```sh
$ keybase pgp gen
```
*¹ When prompted if you want to use a keyphrase when exporting to the **gpg** keychain, remember this decision, it will imply an extra step. I personally don’t set a passphrase when exporting the keys, because my keys don’t leave my computer. New computer, new keys. Also, by opting-out the passphrase setting this step will automatically place the keys in your GPG keyring.*

## Set up Git to sign all commits
```sh
$ gpg --list-secret-keys --keyid-format LONG
# /Users/nacho/.gnupg/pubring.kbx
# -------------------------------
# pub   rsa4096/F0F5C2BDA33D4066 2019-04-27 [SC] [expires: 2035-04-23]
#       E16DBD35E898F5597CE9A770F0F5C2BDA33D4066
# uid                 [ unknown] Ignacio Alvarez <ignacioalvarez92@gmail.com>
# sub   rsa4096/F02705FD89BB1053 2019-04-27 [E] [expires: 2035-04-23]

$ git config --global user.signingkey F0F5C2BDA33D4066
$ git config --global commit.gpgsign true
```

## Add public GPG key to GitHub
```sh
$ keybase pgp export | pbcopy # copy public key to clipboard
$ open https://github.com/settings/keys
# Click "New GPG key"
# Paste key, save
```

## Import key to GPG (omit if you skipped ¹)
```sh
$ keybase pgp export -q F0F5C2BDA33D4066 | gpg --import
$ keybase pgp export -q F0F5C2BDA33D4066 --secret | gpg --allow-secret-key-import --import
```

## Troubleshooting: `gpg failed to sign the data`
If you cannot sign a commit after running through the above steps, and have an error like:

```sh
$ git commit -m "My commit"
# error: gpg failed to sign the data
# fatal: failed to write commit object
```

You can run `echo "test" | gpg --clearsign` to find the underlying issue.

If the above succeeds without error, then there is likely a configuration problem that is preventing git from selecting or using the secret key.  Confirm that your gitconfig `user.email` matches the secret key that you are using for signing.

## Optional: Set as default GPG key
```sh
$ $EDITOR ~/.gnupg/gpg.conf
# Add line:
default-key F0F5C2BDA33D4066
```

## Optional: Configuring gpg binary for git
You may need to configure git to point to the specific gpg executable:
```sh
git config --global gpg.program $(which gpg)
```

## In case you're prompted to enter the username + password

> Most likely related to 2FA being setup in your account.

[Create a Personal Access Token](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line) and, if you are doing this for command line use only, just [x] the **repo** access.

Then, when prompted for username, input yours. When prompted for the password paste the access token.

This will be the last time you'll be prompted for username and password.
