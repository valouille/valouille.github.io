---
title: "How to use KeePassXC with ssh-agent to secure private key access"
date: 2018-03-27T13:37:00+01:00
draft: false
author: VaLouille
type: post
categories:
  - KeePassXC
tags:
  - KeePassXC
  - KeePass
  - KeeAgent
  - macOS
  - SSH
  - key
---

SSH-keys are the most common way to connect to a server securely and in an effortless way. A good practice is to protect the keys with a long-enough passphrase. Since it can be painful to type it every time one wants to login to a server, ssh-agent is often used to bypass this. But this can be a security caveat, since any malware or anybody who can access the laptop can then use the ssh-key to connect to servers.

A good way to prevent this from happening is to use [KeePassXC][1] to manage your ssh-keys. KeePassXC is a password manager, forked from [KeepassX][3], itself a Linux port of [KeePass][4]. KeePassXC is well maintained, and we can take advantage of the new features built inside ! KeePassXC can [now][2] store ssh-keys and associated passphrase, and add them into ssh-agent, allowing SSH connection using public key authentication. It can also unload keys from ssh-agent when the lid is closed, the screen is locked, or in case of prolonged inactivity. And display a confirmation dialog whenever the key is acceded !

# Generate a key pair

If you don't already own a pair of keys, you can use `ssh-keygen` to get new ones.

```
$ ssh-keygen -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/valouille/.ssh/id_rsa):
/Users/valouille/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/valouille/.ssh/id_rsa.
Your public key has been saved in /Users/valouille/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:PGfu6mBVTSTXxMAHx0odNltC8Z0LU61N/xwa+GPgM/M valouille@Valintosh
The key's randomart image is:
+---[RSA 4096]----+
|          .o*XX+o|
|           =ooB*=|
|          ..o=.=+|
|       . . o..+.+|
|        S + o ooo|
|       . = = =  o|
|      o   . * .  |
|     . . .   E   |
|       .o..      |
+----[SHA256]-----+
```

In this case, I create a 4096 bits RSA key. To be able to login as valouille to my server, I add the content of the public key `/Users/valouille/.ssh/id_rsa.pub` inside the `/home/valouille/.ssh/authorized_keys` file.

For now, if I try to connect to my server, I'll be prompted to write down my passphrase to unlock the key. Nothing really new here.

# Enable ssh-agent integration within KeePassXC

In KeePassXC Settings, the checkbox `Enable SSH Agent` from the `SSH Agent` category must be selected. (*A restart of KeePassXC is required*)

{{% figure class="center" src="/images/keepassxc/ssh_agent.png" alt="Enabling ssh agent integration" title="Enabling ssh agent integration" %}}

# Add the key to KeePassXC

After creating a database, we can then add a new entry for the ssh-key :

{{% figure class="center" src="/images/keepassxc/new_entry1.png" %}}

The fields `Password` & `Repeat` are to be filled with the passphrase. Then, we switch to the `SSH Agent` category :

{{% figure class="center" src="/images/keepassxc/new_entry2.png" %}}

The first two checkboxes enable the ssh-agent integration functionality, and the third the dialog window that appears each time the key is used.

From now, ssh-agent should have the key loaded :

```
$ ssh-add -l
4096 SHA256:PGfu6mBVTSTXxMAHx0odNltC8Z0LU61N/xwa+GPgM/M  (RSA)
```

I should be able to login without entering the passphrase, but since I want a dialog window to prompt me whenever my key is used, a few more steps are needed (at least with macOS)

# Making the confirmation dialog window work !

By default, macOS's SSH doesn't ships with an askpass program (like ssh-askpass). This is a pre-requisite for this feature to work. Until then, we get the following error message :

```
$ ssh valouille@server.valouille.fr
sign_and_send_pubkey: signing failed: agent refused operation
valouille@server.valouille.fr: Permission denied (publickey).
```

You can use the following commands to install `ssh-askpass` :

```
git clone https://github.com/theseal/ssh-askpass.git
sudo cp ssh-askpass /usr/local/bin/
cp ssh-askpass.plist ~/Library/LaunchAgents/
```

You'll then need to logout and re-login to enable it.

Once done, when the KeePassXC password database is unlocked, you should be able to login effortlessly :

```
$ ssh server.valouille.fr
```

{{% figure class="center" src="/images/keepassxc/confirmation.png" alt="Prompt to authorize the use of the ssh-key" title="Prompt to authorize the use of the ssh-key" %}}


```
Linux server #1 SMP Debian

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Tue Mar 27 14:59:43 2018
$
```

> To make it easier to validate the dialog window, don't forget to enable the keyboard control from **System Preferences** → **Keyboard** → **Shortcuts** → **Full Keyboard Access...** (at the bottom) → **All Controls**

[1]: https://keepassxc.org/
[2]: https://github.com/keepassxreboot/keepassxc/issues/278
[3]: https://www.keepassx.org/
[4]: https://keepass.info/
