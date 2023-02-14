---
title: Disabling Password Authentication with SSH on macOS Big Sur
date: 2023-02-14 09:02 -0600
categories:
  - system administration
  - macOS
tags:
  - ssh
  - ssh server
  - authentication
---

I recently setup a Mastodon server on Digital Ocean. As part of the instructions
you must disable password authentication via ssh on your server.

On Ubuntu 20.04 this required editing `/etc/ssh/sshd_config`, locating the entry
`PasswordAuthentication`, uncommenting it if it was commented out, and setting
the value to `no`. Then restarting ssh via

```bash
systemctl restart ssh.service
```

Today, I wanted to disable password authentication on a mac server. It's just
about as easy but searching the internet for answers found many old answers that
didn't work. They were instructions for older version of macOS. I finally found
instructions for newer versions (at least for Big Sur) . Here are the
instructions I used that I found at this
[site](https://www.godo.dev/tutorials/macos-ssh-server-no-password/).

Instead of editing `sshd_config` directly, you add a file in
`/ect/ssh/sshd_config.d` with a `.conf` extension to that directory. In that
file add the entries you want to customize. In addition to
`PasswordAuthentication` you must add `ChallengeResponseAuthentication` and set
them both to `no`:

<!-- prettier-ignore-start -->
```text
PasswordAuthentication no
ChallengeResponseAuthentication no
```
{: file="custom.conf" }
<!-- prettier-ignore-end -->

And that's it. You don't even need to restart the ssh server. The changes take
effect immediately.
