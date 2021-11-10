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
*¹ When prompted if you want to use a keyphrase when exporting to the **gpg** keychain, remember this decision, it will imply an extra step. By opting-out the passphrase setting this step will automatically place the keys in your GPG keyring.*

*² If you have other GPG keys indexed by keybase already, potentially from other devices, you can use the `--multi` flag to index a new one.*

## Find your GPG Key ID
Once you created a GPG Key using the step above, find the following
```
$ gpg --list-secret-keys --keyid-format LONG
pub   rsa4096/F0F5C2BDA33D4066 2019-04-27 [SC] [expires: 2035-04-23]
              ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
                   Key ID
```

## Set up Git to sign all commits
```sh
$ git config --global user.signingkey [Your Key ID]
$ git config --global commit.gpgsign true
```

## Add public GPG key to GitHub
```sh
$ keybase pgp export -q [Your Key ID] | pbcopy # copy public key to clipboard
$ open https://github.com/settings/keys
# Click "New GPG key"
# Paste key, save
```

## Import key to GPG (can potentially omit if you skipped¹ and if not elegible for²)
```sh
$ keybase pgp export -q [Your Key ID] | gpg --import
$ keybase pgp export -q [Your Key ID] --secret | gpg --allow-secret-key-import --import
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
default-key [Your Key ID]
```

## Optional: Configuring gpg binary for git
You may need to configure git to point to the specific gpg executable:
```sh
git config --global gpg.program $(which gpg)
```

## In case you're prompted to enter the username + password
### Option 1
[Get a regular SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent). This will be used for authentication/reading/writing to GitHub, not for signing commits, which is taken care of by GPG.

### Option 2
[Create a Personal Access Token](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line) and, if you are doing this for command line use only, just [x] the **repo** access.

Then, when prompted for username, input yours. When prompted for the password paste the access token.
