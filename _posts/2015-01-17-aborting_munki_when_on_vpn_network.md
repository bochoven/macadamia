---
title: Aborting munki when on VPN network
categories:
  - munki
tags:
  - munki
  - munkireport
---

One of the features of the original munkireport that I did not move to munkireport-php was the ability to abort a munki run based on the IP-address of the client machine.
Yesterday Bruno asked if it was possible to use this particular functionality for users that are connecting via a (apparently less capable) VPN.

Munkireport has a possibility to abort runs: add a script to `/usr/local/munki/preflight_abort.d` and if the script returns a nonzero status, the munki run is ended.
 Below you'll find a script (based on the munkiwebadmin preflight script) that will check if the client is on a VPN network. If the client is "inside" the VPN network, the munki run is terminated. 

~~~ bash
#!/bin/bash
 
# The url for your IP-number returning script.
# EXAMPLE: https://example.com/getip.php
LOOKUPURL="http://localhost:9090/getip.php"
 
# List of IP address prefixes from which Munki communication is *denied*.
# EXAMPLE: ( 192.168 172.2 )
VPN_NETWORKS=( 192.168.0.0 127.0)
 
# The optional name of your SSL certificate to use when communicating with
# Your IP server. If you place the certificate in the same directory as this
# script, you don't need to include the full path.
# EXAMPLE: "munkiwebadmin.crt"
MR_SSL_CERTIFICATE=""
 
RUNTYPE="$1"
 
if [ "$RUNTYPE" == "custom" -o "$RUNTYPE" == "auto" ]; then
    if [ ! -z "$VPN_NETWORKS" ]; then
        echo "Checking network"
 
        paramList=" --silent --max-time 5 --fail"
 
        if [ ! -z "${MR_SSL_CERTIFICATE}" ]
		then
			# a cert was specified, can we find it?
			if [ -e "${MR_SSL_CERTIFICATE}" ]
			then
				paramList="${paramList} --cacert ${MR_SSL_CERTIFICATE}"
			else
				echo "Cannot locate cert named '${MR_SSL_CERTIFICATE}'"
			fi
		fi
 
        external_ip=`/usr/bin/curl ${paramList} "${LOOKUPURL}"`
 
        if [ $? -ne 0 ]; then
            echo "External IP lookup failed, aborting munki run"
            exit 1
        fi
         
        FOUND="no"
        for prefix in ${VPN_NETWORKS[@]}; do
            echo -n "Checking $prefix for IP address $external_ip"
            prefix_len=$( echo `echo "$prefix" | tr . '\012' | wc -l` )
            for (( i=$prefix_len ; i<4 ; i++ )); do
                echo -n '.x'
            done
            my_ip_prefix=`echo $external_ip | cut -d. -f1-$prefix_len`
            if [ "$my_ip_prefix" == "$prefix" ]; then
                echo ": Inside VPN"
                FOUND="yes"
                break
            else
                echo ": Outside VPN"
            fi
        done
        if [ "$FOUND" == "no" ]; then
            echo "Network check OK, proceeding with munki run"
        else
            echo "Network check failed, aborting munki run"
            exit 1
        fi
    fi
fi
~~~

The above script assumes there is a server side script that tells the client which IP address it is using (getip.php). Below is the php script that needs to be placed on the munkireport server (next to index.php)

~~~ php
<?php
	// Return Remote IP address as seen from the server
	
	// Test for proxy
	if(isset($_SERVER["HTTP_X_FORWARDED_FOR"]))
	{
		echo $_SERVER["HTTP_X_FORWARDED_FOR"];
	}
	else
	{
		echo $_SERVER['REMOTE_ADDR'];
	}
?>
~~~