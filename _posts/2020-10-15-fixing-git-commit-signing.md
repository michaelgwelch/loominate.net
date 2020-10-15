# Fixing git Commit Signing

Today I went to commit something to a repo and this happened:

```bash
$ git commit
error: gpg failed to sign the data
fatal: failed to write commit object
```

I know that I successfully committed something yesterday and couldn't think of any tooling changes since then.

I attempted a few things without success:

* Changed the format of my signing key in my `.gitconfig` file per [Troubleshooting gpg git commit signing](https://juliansimioni.com/blog/troubleshooting-gpg-git-commit-signing/).
* Rebooted
* Checked for recent updates to `git`, `brew` etc.

Then I fired up *GPG Keychain* the application I use for managing my keys on my Mac. It promptly informed me that my key expired today and offered to extend the key for me. I recalled I had already extended my key on my other computer so I exported it from one computer and imported it to my other computer and my ability to sign git commits was restored.