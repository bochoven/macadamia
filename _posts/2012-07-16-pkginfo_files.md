---
title: "Munki pkg: Reposado installer"
categories:
  - munki
tags:
  - munki
  - reposado
  - IIS
---


I'm running reposado from a Windows (IIS) server, and I'm not able to run repo_sync or repoutil on the server itself. But I *can* mount the reposado share on a client.
The code below installs the latest version of reposado using git. The pkginfo file needs another pkg that installs git on the machine and depends on that.
It also configures reposado by writing the pref file. 
You'll have to change

* BASEURL=https://reposado.myinstitute.edu
* METADIR=/tmp/SU/metadata
* UPDATEDIR=/tmp/SU/www

to your own paths in the below script

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>autoremove</key>
	<false/>
	<key>catalogs</key>
	<array>
		<string>production</string>
	</array>
	<key>name</key>
	<string>reposado</string>
	<key>installer_type</key>
	<string>nopkg</string>
	<key>installs</key>
	<array>
		<dict>
			<key>path</key>
			<string>/usr/local/reposado</string>
			<key>type</key>
			<string>file</string>
		</dict>
	</array>
	<key>preinstall_script</key>
	<string>#!/bin/bash

#VARIABLES
BASEURL=https://reposado.myinstitute.edu
METADIR=/tmp/SU/metadata
UPDATEDIR=/tmp/SU/www

echo 'Cloning reposado'
cd /usr/local/
git clone https://github.com/wdas/reposado.git

echo 'Setting prefs'
echo '&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"&gt;
&lt;plist version="1.0"&gt;
&lt;dict&gt;
&lt;key&gt;LocalCatalogURLBase&lt;/key&gt;
&lt;string&gt;$BASEURL&lt;/string&gt;
&lt;key&gt;UpdatesMetadataDir&lt;/key&gt;
&lt;string&gt;$METADIR&lt;/string&gt;
&lt;key&gt;UpdatesRootDir&lt;/key&gt;
&lt;string&gt;$UPDATEDIR&lt;/string&gt;
&lt;/dict&gt;
&lt;/plist&gt;' &gt; /usr/local/reposado/code/preferences.plist

echo 'Adding reposado to search path'
echo /usr/local/reposado/code &gt; /etc/paths.d/reposado

exit 0</string>
	<key>uninstallable</key>
	<true/>
	<key>uninstall_method</key>
	<string>uninstall_script</string>
   <key>uninstall_script</key>
   <string>#!/bin/sh
rm -rf "/usr/local/reposado"
rm "/etc/paths.d/reposado"
   </string>
   <key>version</key>
   <string>1.0.0</string>
   <key>requires</key>
   <array>
      <string>git</string>
   </array>
</dict>
</plist>
~~~