---
title: Creating a custom munki installer
categories:
  - munki
tags:
  - munki
---

In my organization, we offer munki as a self-service package that users can install themselves. This package contains the 'standard' munki installer plus the required settings to make the munki client connect to our munki server.

To include the settings we're using [munki-rebrand](https://github.com/ox-it/munki-rebrand), a customization script that I wrote in 2014 and that got adopted by the fine people at Oxford IT. Recently [Ben Goodstein](https://github.com/fuzzylogiq) rewrote the  script and the current script is infinitely better than the original.

### Prerequisites

You'll need a Mac with Xcode installed and no fear of the Terminal.

### Get on with it

First, we create our build directory and put the rebrand script inside. I'm building in `/Users/Shared`, but you can pick any directory. Open Terminal.app and start typing (or copy/paste).

~~~ bash
cd /Users/Shared
git clone https://github.com/ox-it/munki-rebrand.git munki_installer
~~~

Now we create our postinstall script which will set up our settings. In my org we only set `SoftwareRepoURL` and `ClientIdentifier`, but you can add any setting you need. 

~~~ bash
cd munki_installer

echo '#!/bin/bash' > postinstall.sh
echo 'SETPREF="defaults write /Library/Preferences/Managedinstalls"' >> postinstall.sh
echo '$SETPREF SoftwareRepoURL https://munki.my.org/repo' >> postinstall.sh
echo '$SETPREF ClientIdentifier selfservice' >> postinstall.sh
~~~

Now we can build the package. To be safe, we use the `tag` of the [latest munki](https://github.com/munki/munki/releases/latest) release ('v2.8.2' at the moment). We need root privileges so we start the command with `sudo`

~~~ bash
sudo ./munki_rebrand.py --postinstall='postinstall.sh' --munki-release='v2.8.2' --appname='Managed Software Center'
~~~

If you end up with an installer package, good job! If not, retrace your steps and see what went wrong. You can distribute this package with your favorite package distribution tool.

### Sign the package

To make the package play nice with macOS GateKeeper, you can sign the package with your developer account. Make sure you have the developer certificate installed in your keychain and run this command (replace the sign string with your own):

~~~ bash
productsign --sign "Developer ID Installer: Your Org (X00XX0XX0X)" munkitools-2.8.2.2855.pkg munkitools-2.8.2.2855-signed.pkg
~~~

We put up our installer package on an intranet page with some instructions for the end user, but you can use whatever you need to get the package on the machines.

### Further customization options

So far we only covered adding a postinstall script, the rebrand script can also rename the App (which was the original purpose) and add a custom icon. Please refer to the [munki-rebrand github page](https://github.com/ox-it/munki-rebrand) for more information.

## Background

The default [munki](https://github.com/munki/munki/releases/latest) installer package installs munki with some default settings that may not fit your organization:

  * SoftwareRepoURL: 'http://munki/repo'
  * ClientIdentifier: ''
  * InstallAppleSoftwareUpdates: `False`,
  * UseClientCertificate: `False`,
  * etc.

There are several ways to add your required settings to a munki install: 

* deploy a seperate 'settings package' containing a configuration profile or a script with a couple of `defaults` commands
* have a server listening on http://munki/repo with a site_default manifest that contains a package that sets up the config for the real munki server.
* disassemble the munki metapkg and re-assemble with an added settings package.

These methods have their pros and cons, in my org, we opted to go a different route.

