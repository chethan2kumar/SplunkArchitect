# Splunk Components Quick Reference Checklist

### Contents <a name="toc"></a>

1. [Installing Splunk Enterprise on Linux](#install_splunk)
2. [Uninstalling Splunk Enterprise on Linux](#uninstall_splunk)
3. [Create a Splunk Indexer](#create_indexer)
4. [Create a Cluster Master](#create_cluster_master)
10. [Install a Universal Forwarder](#install_uf)



## Installing Splunk Enterprise on Linux <a name="install_splunk"></a>

Access the Splunk website to obtain the latest Splunk Enterprise installation package:
<a href="https://www.splunk.com/en_us/download/sem.html?ac=ga_usa_brand_enterprise_exact_Mar17&_kk=splunk%2520enterprise&gclid=CIvWzN6Hk9MCFQsRgQodK_QARg" target="_blank">Download Splunk Enterprise RPM for Linux-x86_64</a>
__Create an account and/or login using your Splunk user ID & password.__

To fetch the splunk installable directly from a target server, use wget:  
* Select the appropriate Splunk download from the website link above
* Cancel the download window
* From the page that appears after the download appears, in the top-right click the link in 'Download via Command Line (wget)'
* Copy the wget string and paste it into a command shell  

Example wget string:  
```wget -O splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.3&product=splunk&filename=splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm&wget=true'```
or...
FTP or WinSCP install package to target server(s)  
<a href="https://winscp.net/eng/download.php" target="_blank">WinSCP Download</a>

__All Splunk components__

sudo su
chmod 744 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm  
rpm -i splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm  
cd /opt/splunk/bin  
./ splunk start --accept-license  
./ splunk enable boot-start  
cd /ect  
vi profile  (add and then run:)
export SPLUNK_HOME="/opt/splunk"

__All Splunk components except Indexers and Universal Forwarders:__

```/opt/splunk/etc/system/local/outputs.conf```
```
# Turn off indexing (except on indexers)
[indexAndForward]
index = false
 
[tcpout]
defaultGroup = my_search_peers 
forwardedindex.filter.disable = true  
indexAndForward = false 
 
[tcpout:my_search_peers]
server=10.10.10.1:9997,10.10.10.2:9997,10.10.10.3:9997
autoLB = true
```

__Login to Splunk Web__

```http://<yourservername>:8000/```
admin - changeme - MyNewPassword  

## Uninstalling Splunk Enterprise on Linux <a name="uninstall_splunk"></a>

Get package name:
```rpm -q -a | grep -i splunk```

Uninstall:
```rpm -e <packageName>```
Example:
```rpm -e splunk-6.5.3-36937ad027d4.x86_64```

Clean up remaining files/directory:
```cd /opt```
```rm -rf ./splunk```


## Create a Splunk Indexer <a name="create_indexer"></a>

Settings > Indexer clustering > Enable indexer clustering > Peer node > Next

* Master URI. Enter the master's URI, including its management port. For example: https://10.152.31.202:8089.
* Peer replication port. This is the port on which the peer receives replicated data streamed from the other peers. You can specify any available, unused port for this purpose. This port must be different from the management or receiving ports.
* Security key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. Set the same value here that you previously set on the master node.

Click Enable peer node & Restart Splunk  

When you have enabled the 'replication factor' number of peers, the cluster can start indexing and replicating data (it will be blocked by the master node until the RF number of peers is enabled).

__Configure peer nodes with server.conf__

```/opt/splunk/etc/system/local/server.conf```
```
[replication_port://9887]

[clustering]
master_uri = https://10.152.31.202:8089
mode = slave
pass4SymmKey = whatever
```

## Create a Cluster Master <a name="create_cluster_master"></a>

__To configure a Splunk Enterprise instance as the Cluster Master / master node:__

Settings > Indexer clustering > Enable indexer clustering > Master node > Next

* Replication Factor. The replication factor determines how many copies of data the cluster maintains. The default is 3. Be sure to choose the right replication factor now. It is inadvisable to increase the replication factor later, after the cluster contains significant amounts of data.  
* Search Factor. The search factor determines how many immediately searchable copies of data the cluster maintains. The default is 2. Be sure to choose the right search factor now. It is inadvisable to increase the search factor later, once the cluster has significant amounts of data.  
* Security Key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. The value that you set here must be the same that you subsequently set on the peers and search heads as well.  
* Cluster Label. You can label the cluster here. The label is useful for identifying the cluster in the monitoring console. See Set cluster labels in Monitoring Splunk Enterprise.

Enable master node & Restart Splunk
Also:
[Forward internal data to search peers](#fwd_internal_data)

When the master starts up for the first time, it will block indexing on the peers until you enable and restart the full replication factor number of peers. Do not restart the master while it is waiting for the peers to join the cluster. If you do, you will need to restart the peers a second time.

After the restart, log back into the master and return to the Clustering page in Splunk Web (Settings > Indexer Clustering>). You will see the master clustering dashboard. 

__Configuring a Cluster Master using server.conf__

Enabling a Cluster Master or editing its configuration can also be done in ```/opt/splunk/etc/system/local/server.conf```:
```
[clustering]
mode = master
replication_factor = 4
search_factor = 3
pass4SymmKey = whatever
cluster_label = cluster1
```

### Configuring indexes.conf on a Cluster Master <a name="configuring_indexes_cluster"></a>

If you are using a index cluster, you must configure custom indexes in an indexes.conf file on the Cluster Master in a ```/opt/splunk/etc/master-apps/<app-name>/local/``` directory.

The following is an example of typical custom index entries in an indexes.conf file:  
```
[<index_name>]
homePath   = volume:primary/index_name/db
coldPath   = volume:primary/index_name/colddb
thawedPath = $SPLUNK_DB/index_name/thaweddb
# Seting the repFactor attribute to "auto" causes the index's data to be replicated to other peers in the cluster
# By default, repFactor is set to 0, which means that the index will not be replicated
repFactor = auto
# index size of 30 GB = 30 x 1024 MB
maxTotalDataSizeMB = 30720
# only keep data for 30 days before archiving (default is 6 years)
frozenTimePeriodInSecs = 2592000
# you can specify a frozen archive location
# if no fozen archive location is specified, the data is discarded
# coldToFrozenDir = "<path to frozen archive>"
```

### Configuring props.conf files <a name="props_conf"></a>

The props.conf (and transforms.conf, if applicable) file is distributed across the indexers ('search peers') to tell Splunk how to parse incoming data. You configure props.conf & transforms.conf in the ```/opt/splunk/etc/master-apps/<app-name>/local/``` directories. 

When the cluster bundle is applied (distributed to the indexers / search peers) these same files will be located in ```/opt/splunk/etc/apps/<app-name>/local/``` directories on each indexer.

Here is an assorted example of some props.conf entries; check the options for each of these entries in the [props.conf.example](./README/props.conf.example) or [props.conf.spec](./README/props.conf.spec) files:
```
BREAK_ONLY_BEFORE = ([\r\n]+)
DATETIME_CONFIG = 
MAX_TIMESTAMP_LOOKAHEAD = 30
NO_BINARY_CHECK = true
TIME_FORMAT = %Y-%m-%dT%H:%M:%S.%3N
TIME_PREFIX = timestamp="
TRUNCATE = 0
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)\[\d{4}-\w{3}-\d{1,2}\s\d{1,2}:\d{1,2}:\d{1,2}\.\d{1,3}\]\s\w+
PREAMBLE_REGEX = ^log4j\:[^\r\n]+[\r\n][^:]+[^\s]+[^\r\n]
KV_MODE = json

# Used with a transforms.conf file
# ID the sourcetype to perform the transform on:
[sourcetype::mssqlserver]
# Then TRANSFORMS-<arbitrary name> = <stanza name in transforms.conf>
TRANSFORMS-DBHost = db_host
```

### Configuring transforms.conf files <a name="transforms.conf"></a>

The transforms.conf file is used to 

Here are some assorted samples of transforms.conf entires; check the options for each of these entries in the [transforms.conf.example](./README/transforms.conf.example) or [transforms.conf.spec](./README/transforms.conf.spec) files:

```
# Used with the props.conf file in the previous example
# [db_host] matches with `TRANSFORMS-DBHost = db_host' in props.conf
[db_host]
REGEX = MachineName="(?<hostname>.*?)"
# Note that 'hostname' must match the RegEx field name above
FORMAT = hostname::$1
# use MetaData:Host to replace the default host entry in an event
DEST_KEY = MetaData:Host


# Another example
[get_path]
SOURCE_KEY = uri
REGEX = ^(?<path>[^\s\?]+) 

[get_query_string]
SOURCE_KEY = uri
REGEX = \?(<query_string>[^\s]+) 
```

__To distribute a new configuration bundle: __

* Validate the bundle: ```splunk validate cluster-bundle```
* You can check the status of bundle validation ```splunk show cluster-bundle-status```
* Apply the bundle: ```splunk apply cluster-bundle```
* To avoid having to answer the prompt: ```splunk apply cluster-bundle --answer-yes```

__NOTE: Be sure to validate the cluster bunde before running ```splunk apply``` - if you apply a bundle with errrors in the indexes.conf file you could bring the entire indexer cluster down, necessitating fixing the error manually on each indexer and restarting it.__

## Create a Splunk Search Head <a name="create_search_head"></a>

__Configure a Splunk instance as a search head in an indexer cluster:__

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Distributed search 
3. 


	The message appears, "You must restart Splunk for the search node to become active. You can restart Splunk from Server Controls."

7. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.  

8. After the restart, log back into the search head and return to the Clustering page in Splunk Web. This time, you see the search head's clustering dashboard.  




## Install a Universal Forwarder <a name="install_uf"></a>

<a href="https://www.splunk.com/en_us/download/universal-forwarder.html#tabs/linux" target="_blank">Download Splunk Universal Forwarder</a>

__Note: You will have to create an account and/or login using your Splunk user ID & password.__

* Select and download the appropriate rpm package (linux > 64-bit > .rpm)
* Use ftp or WinSCP to copy the package to the splunk server.  
* ```sudo su```  (root)
```chmod 744 splunkforwarder-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm```
```rpm -i splunkforwarder-<…>-linux-2.6-x86_64.rpm```
```cd /opt/splunkforwarder/bin/```
```./splunk start --accept-license```



> Written with [StackEdit](https://stackedit.io/).