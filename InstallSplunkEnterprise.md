
# Install Splunk Enterprise

### Contents <a name="toc"/>

1. [Installing Splunk Enterprise on Linux](#install_splunk)
	* [Obtain the Splunk installation package](#get_package)
	* [Installing the Splunk package](#install_package)
	* [Logging into Splunk Enterprise - First Time](#login)
	* [Post Installation Settings](#post_install)
	* [Disable Transparent Huge Pages](#disable_thp)
	* [Set the environment variable for $SPLUNK_HOME](#set_env)
	* [Change the default Splunk Web port from 8000 to 8080 (Optional)](#set_port_8080)
	* [Configure Splunk Web to use SSL (Recommended)](#set_ssl)

2. [Create a Splunk Search Head](#create_search_head)



## Installing Splunk Enterprise on Linux <a name="install_splunk"/>
All Splunk components except a Universal Forwarder ( a separate lightweight package ) are based on an installation of Splunk Enterprise with specific configuration options.

### Obtain the Splunk installation package <a name="get_package"/>

* Access the Splunk website:  
<a href="https://www.splunk.com/en_us/download/sem.html?ac=ga_usa_brand_enterprise_exact_Mar17&_kk=splunk%2520enterprise&gclid=CIvWzN6Hk9MCFQsRgQodK_QARg" target="_blank">Download Splunk Enterprise RPM for Linux-x86_64</a>

__Note: You will have to create an account and/or login using your Splunk user ID & password.__

* Select and download the appropriate rpm package (...x86_64.rpm)
* Use ftp or WinSCP to copy the package to the splunk server.  
* Change permissions on the file:
previous:  
```-rw-------. 1 baxtj018 admin 226438185 Apr  7 15:13 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm```

	```chmod 744 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm```

### Installing the Splunk package <a name="install_package"/>

```sudo rpm -i splunk-6.5.1-f74036626f0c-linux-2.6-x86_64.rpm  ```

You may see:  
warning: splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY  
then:  
complete  

* Start splunk:
	```
	sudo su
	cd /opt/splunk/bin  
	./splunk start  
	```  
    
* Space-down and answer 'y' to the license question, or use:  

```./splunk start --accept-license```

You'll see a series of startup messages, then:  

```The Splunk web interface is at http://<yourservername>:8000```

### Logging into Splunk Enterprise - First Time <a name="login"/>

* Login to Splunk Web: ```http://<yourservername>:8000/```
```
user: admin
password: changeme
(this is the default password upon a new installation)

New password: MyNewPassword
```
You may see a 'Help us improve Splunk software' message...  

* un-check the 'Share license usage data with Splunk' box  and click 'OK'  

### Post Installation Settings <a name="post_install"/>

These are recommended settings to ensure that Splunk starts automatically after a server reboot, set the default environment variable, and 

__Note: all splunk commands are run from /opt/splunk/bin__

* Configure linux to start Splunk on boot:

```./splunk enable boot-start```  

You'll get messages:  
Init script installed at /etc/init.d/splunk.
Init script is configured to run at boot.

The enable boot-start exercise above creates a 'splunk' start-up script file in /etc/rc.d/init.d

#### Disable Transparent Huge Pages <a name="disable_thp"/>

The /etc/rc.d/init.d file should be edited to add functions to disable Transparent Huge Pages on some linux distros such as Red Hat.

<a href="https://github.com/packetiq/SplunkArchitect/blob/master/DisableTransparentHugePages.md" target="_blank">Instructions</a>

#### Set the environment variable for $SPLUNK_HOME <a name="set_env"/>

Append this to the bottom of /etc/profile file:

```export SPLUNK_HOME="/opt/splunk"```

and run this command from the command line as well to create the variable in the current session.  

You can now do

```$SPLUNK_HOME/bin/<command>```
or
```cd $SPLUNK_HOME```

#### Change the default Splunk Web port from 8000 to 8080 (Optional) <a name="set_port_8080"/>

For an initial test scenario, you may want to use non-SSL access, on a non-standard port. __See the next section to configure a different port and/or use SSL__

```/opt/splunk/bin/splunk set web-port 8080```

This adds the following to opt/splunk/etc/system/local/web.conf

	[settings]
	httpport = 8080  


### Configure Splunk Web to use SSL (Recommended) <a name="set_ssl"/>

From Splunk Web:

Settings > Server settings > General settings 
Splunk Web

Run Splunk Web: Yes (on by default)
Enable SSL (HTTPS) in Splunk Web?: Yes
Web port: 8443
Click Save
This will modify /opt/splunk/etc/system/local/web.conf to contain:
```
[settings]
httpport = 8443
enableSplunkWebSSL = true
```

__You are now ready to customize this instance of Splunk Enterprise for a specific function__

The following sections outline how to configure instances of Splunk Enterprise to perform specific functions (search head, indexer, etc.) in a distributed and/or clustered Splunk solution.   

If you are using a standalone instance of Splunk Enterprise you may want to apply some of the function-specific settings outlined in the following sections. Generic settings that may be applied to any Splunk solution will be indicated.

[top](#toc)

## Create a Splunk Search Head <a name="create_search_head"/>



[top](#toc)

## Create a Splunk Indexer 
[top](#toc)

## Create a Deployment Server

[top](#toc)

## Install a Universal Forwarder

[top](#toc)

## Splunk Common Network Ports

![Splunk Common Network Ports](369-splunk-common-network-ports-ver1.5.jpg)  

> Written with [StackEdit](https://stackedit.io/).