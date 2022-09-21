---
title: "Use KeePassXC to automatically manage website passwords"
date: 2018-03-29T13:37:00+01:00
draft: true
author: VaLouille
type: post
categories:
  - KeePassXC
tags:
  - KeePassXC
  - KeePass
  - KeeAgent
  - macOS
  - website
  - passwords
  - Chrome
  - Chromium
  - Firefox
---

[KeePassXC][1] can be used to store and autofill username and passwords for websites. KeePassXC is a password manager, forked from [KeepassX][3], itself a Linux port of [KeePass][4]. KeePassXC is well maintained, and supports Firefox, Google Chrome, Chromium and Vivaldi. It's open-source, which is good for a security tool, and supports Windows, Linux & macOS. This means that you can sync your passwords for example between the Chrome of your MacBook and the Firefox of your Windows PC. A cross-platform cloud file sharing tool like [Owncloud][5] or [Pydio][6] is needed to update automatically the database file on several machines.

# Browser plugin installation

A browser plugin is needed for KeePassXC integration. The plugins are available for [Chrome, Chromium  & Vivaldi][7] and for [Firefox][8]. Once the plugin is installed, a little icon like this will appear :

{{% figure class="left" src="/images/keepassxc/extension_disabled.png" alt="Extension not yet configured" title="Extension not yet configured" %}}

# Configuration of KeePassXC

To enable the browser integration feature, in the settings of KeePassXC, the `Enable KeePassXC browser integration` checkbox must be checked, and the checkbox for each browser also.

{{% figure class="center" src="/images/keepassxc/enable_browser_integration.png" alt="Enabling browser integration" title="Enabling browser integration" %}}

## Fix the (possible) error with Chrome

With Chrome, I faced the following error when saving the parameters :

```
Cannot save the native messaging script file.
```

To fix this issue, I manually (with the sudo permission) created the `org.keepassxc.keepassxc_browser.json` file with the following content :

```
{
    "allowed_origins": [
        "chrome-extension://oboonakemofpalcgghocfoadofidjkkk/"
    ],
    "description": "KeePassXC integration with native messaging support",
    "name": "org.keepassxc.keepassxc_browser",
    "path": "/Applications/KeePassXC.app/Contents/MacOS/keepassxc-proxy",
    "type": "stdio"
}
```

The file should be present at the following location :
* Chrome: `~/Library/Application Support/Google/Chrome/NativeMessagingHosts`
* Chromium: `~/Library/Application Support/Chromium/NativeMessagingHosts`
* Vivaldi: `~/Library/Application Support/Vivaldi/NativeMessagingHosts`

# Usage of the plugin and KeePassXC

## Adding a password to KeePassXC from the Browser

## Manually adding a password to KeePassXC


[1]: https://keepassxc.org/
[2]: https://addons.mozilla.org/en-US/firefox/addon/keepassxc-browser/
[3]: https://www.keepassx.org/
[4]: https://keepass.info/
[5]: https://owncloud.org/
[6]: https://pydio.com/
[7]: https://chrome.google.com/webstore/detail/keepassxc-browser/oboonakemofpalcgghocfoadofidjkkk
[8]: https://addons.mozilla.org/en-US/firefox/addon/keepassxc-browser/
