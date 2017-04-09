
# Install and Configure Splunk Enterprise Components

### Contents <a name="toc"></a>

1. [Installing Splunk Enterprise on Linux](#install_splunk)
	* [Obtain the Splunk installation package](#get_package)
	* [Installing the Splunk package](#install_package)
	* [Logging into Splunk Enterprise - First Time](#login)
	* [Post Installation Settings](#post_install)
	* [Disable Transparent Huge Pages](#disable_thp)
	* [Set the environment variable for $SPLUNK_HOME](#set_env)
	* [Change the default Splunk Web port from 8000 to 8080 (Optional)](#set_port_8080)
	* [Configure Splunk Web to use SSL (Recommended)](#set_ssl)
	* [Synchronize system clocks across the distributed search environment](#clocks) 

2. [Overview of a Clustered Splunk Environment](#clustered_overview)
	* [Indexer Clustering](#indexer_clustering)


3. [Create a Splunk Indexer](#create_indexer)
	* [Managing Configurations Across Peers](#managing_peer_configs)
	* [Multisite indexer clusters](#multisite_clusters)
	
4. [Create a Cluster Master](#create_cluster_master)
	* [Configuring Indexes on a Cluster](#configuring_indexes_cluster)

5. [Create a Splunk Search Head](#create_search_head)
	* [Index Time vs Search Time Processing](#index_vs_search)
	* [Distributed Search](#distributed_search)

6. [Create a Deployment Server](#create_deployment_server)

7. [Create a Deployer](#create_deployer)

8. [Create a License Server](#create_lic_svr)
	* [Licenses for distributed search](#search_licenses)

9. [Install a Universal Forwarder](#install_uf)

10. [Splunk Common Network Ports](#network_ports)


## Installing Splunk Enterprise on Linux <a name="install_splunk"></a>
All Splunk components except a Universal Forwarder ( a separate lightweight package ) are based on an installation of Splunk Enterprise with specific configuration options - so the first step in creating any component in a Splunk solution is installing Splunk Enterprise.  

### Obtain the Splunk installation package <a name="get_package"></a>

* Access the Splunk website:  
<a href="https://www.splunk.com/en_us/download/sem.html?ac=ga_usa_brand_enterprise_exact_Mar17&_kk=splunk%2520enterprise&gclid=CIvWzN6Hk9MCFQsRgQodK_QARg" target="_blank">Download Splunk Enterprise RPM for Linux-x86_64</a>

__Note: You will have to create an account and/or login using your Splunk user ID & password.__

* Select and download the appropriate rpm package (...x86_64.rpm)
* Use ftp or WinSCP to copy the package to the splunk server.  
* Change permissions on the file:
previous:  
```-rw-------. 1 baxtj018 admin 226438185 Apr  7 15:13 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm```

	```chmod 744 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm```

### Installing the Splunk package <a name="install_package"></a>

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

### Logging into Splunk Enterprise - First Time <a name="login"></a>

* Login to Splunk Web: ```http://<yourservername>:8000/```
```
user: admin
password: changeme
(this is the default password upon a new installation)

New password: MyNewPassword
```
You may see a 'Help us improve Splunk software' message...  

* un-check the 'Share license usage data with Splunk' box  and click 'OK'  

### Post Installation Settings <a name="post_install"></a>

These are recommended settings to ensure that Splunk starts automatically after a server reboot, set the default environment variable, and 

__Note: all splunk commands are run from /opt/splunk/bin__

* Configure linux to start Splunk on boot:

```./splunk enable boot-start```  

You'll get messages:  
Init script installed at /etc/init.d/splunk.
Init script is configured to run at boot.

The enable boot-start exercise above creates a 'splunk' start-up script file in /etc/rc.d/init.d

#### Disable Transparent Huge Pages <a name="disable_thp"></a>

The /etc/rc.d/init.d file should be edited to add functions to disable Transparent Huge Pages on some linux distros such as Red Hat.

<a href="https://github.com/packetiq/SplunkArchitect/blob/master/DisableTransparentHugePages.md" target="_blank">Instructions</a>

#### Set the environment variable for $SPLUNK_HOME <a name="set_env"></a>

Append this to the bottom of /etc/profile file:

```export SPLUNK_HOME="/opt/splunk"```

and run this command from the command line as well to create the variable in the current session.  

You can now do

```$SPLUNK_HOME/bin/<command>```
or
```cd $SPLUNK_HOME```

#### Change the default Splunk Web port from 8000 to 8080 (Optional) <a name="set_port_8080"></a>

For an initial test scenario, you may want to use non-SSL access, on a non-standard port. __See the next section to configure a different port and/or use SSL__

```/opt/splunk/bin/splunk set web-port 8080```

This adds the following to opt/splunk/etc/system/local/web.conf

	[settings]
	httpport = 8080  


### Configure Splunk Web to use SSL (Recommended) <a name="set_ssl"></a>

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

#### Synchronize system clocks across the distributed search environment <a name="clocks"></a>

Synchronize the system clocks on all machines, virtual or physical, that are running Splunk Enterprise distributed search instances. Specifically, this means your search heads and search peers. In the case of search head pooling or mounted bundles, this also includes the shared storage hardware. Otherwise, various issues can arise, such as bundle replication failures, search failures, or premature expiration of search artifacts.

The synchronization method that you use depends on your specific set of machines. For most environments, Network Time Protocol (NTP) is the best approach.

__You are now ready to customize this instance of Splunk Enterprise for a specific function__

The following sections outline how to configure instances of Splunk Enterprise to perform specific functions (search head, indexer, etc.) in a distributed and/or clustered Splunk solution.   

It may be helpful to read the section '[Overview of a Clustered Splunk Environment](#clustered_overview)' below before progressing to specific component configuration.

If you are using a standalone instance of Splunk Enterprise you may want to apply some of the function-specific settings outlined in the following sections. Generic settings that may be applied to any Splunk solution will be indicated.

[top](#toc)

## Overview of a Clustered Splunk Solution <a name="clustered_overview"></a>

### Indexer Clustering <a name="indexer_clustering"></a> 

Splunk supports the concept of utilizing multiple indexers in an 'index cluster' to increase indexing capability and to allow setting a 'replication factor' which creates multiple, redundant copies of rawdata files across multiple indexers so that if an indexer fails data is not lost, as well as a 'search factor' wherein multiple copies of data indexes are distributed as well so that search performance is not affected if an indexer fails.

The image below depicts a simple clustered indexer solution:

![Clustered Indexer Solution](/images/Simplified_basic_cluster_60.png)  

There are two types of nodes in an indexer cluster:

*  A single Cluster Master to manage the cluster. This 'master node' is a specialized type of indexer that, while it doesn't participate in any data streaming, coordinates a range of activities involving the search peers and the search head, including coordinating which 'buckets' are replicated across the peer nodes for redundancy.

* Several 'peer nodes' (indexers) that handle the indexing function for the cluster, indexing and maintaining multiple copies of the rawdata and index files, and running searches across the data.

#### Index Time vs Search Time Processing <a name="index_vs_search"></a>

Splunk Enterprise documentation contains references to the terms "index time" and "search time". These terms distinguish between the types of processing that occur during indexing, and the types that occur when a search is run.

It is better to perform most knowledge-building activities, such as field extraction, at search time. Index-time custom field extraction can degrade performance at both index time and search time.

Index-time processes take place between the point when the data is consumed and the point when it is written to disk.

The following processes occur during index time (controlled by __props.conf__ on an indexer):

* Default field extraction (such as host, source, sourcetype, and timestamp)
* Static or dynamic host assignment for specific inputs
* Default host assignment overrides
* Source type customization
* Custom index-time field extraction
* Structured data field extraction
* Event timestamping
* Event linebreaking
* Event segmentation (also happens at search time)

Search-time processes take place while a search is run, as events are collected by the search. The following processes occur at search time:

* Event segmentation (also happens at index time)
* Event type matching
* Search-time field extraction (automatic and custom field extractions, including multivalue fields and calculated fields)
* Field aliasing
* Addition of fields from lookups
* Source type renaming
* Tagging  

[top](#toc)

## Create a Splunk Indexer <a name="create_indexer"></a>

Splunk Enterprise stores all of the data it processes in indexes. An index is a collection of databases, which are subdirectories located in ```$SPLUNK_HOME/var/lib/splunk```.  

Indexes consist of two types of files: compressed __rawdata files__ and __index files__. 

Splunk Enterprise comes with a number of preconfigured indexes, including:  

* main: This is the default Splunk Enterprise index. All processed data is stored here unless otherwise specified.
* _internal: Stores Splunk Enterprise internal logs and processing metrics.
* _audit: Contains events related to the file system change monitor, auditing, and all user search history.

__To enable an indexer as a peer node:__

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable indexer clustering.
4. Select Peer node and click Next.
5. There are a few fields to fill out:

* Master URI. Enter the master's URI, including its management port. For example: https://10.152.31.202:8089.

* Peer replication port. This is the port on which the peer receives replicated data streamed from the other peers. You can specify any available, unused port for this purpose. This port must be different from the management or receiving ports.

* Security key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. Set the same value here that you previously set on the master node.

6. Click Enable peer node.

The message appears, "You must restart Splunk for the peer node to become active."

7. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.

8. Repeat this process for all the cluster's peer nodes.

When you have enabled the 'replication factor' number of peers, the cluster can start indexing and replicating data (it will be blocked by the master node until the RF number of peers is enabled).

__Configure peer nodes with server.conf__

You can configure an indexer / peer node my editing the server.conf file in ```/opt/splunk/etc/system/local/```; note the 'mode = slave' entry:

```
[replication_port://9887]

[clustering]
master_uri = https://10.152.31.202:8089
mode = slave
pass4SymmKey = whatever
```
This example specifies that:

* the peer will use port 9887 to listen for replicated data streamed from the other peers. You can specify any available, unused port as the replication port. Do not re-use the management or receiving ports.
* the peer's cluster master resides at 10.152.31.202:8089.
* the instance is a cluster peer ("slave") node.
* the security key is "whatever".

### Multisite indexer clusters <a name="multisite_clusters"></a>

Multisite indexer clusters allow you to maintain complete copies of your indexed data in multiple locations. This offers the advantages of enhanced disaster recovery and search affinity. You can specify the number of copies of data on each site. Multisite clusters are similar in most respects to basic, single-site clusters, with some differences in configuration and behavior. 

For multisite clusters, you must also take into account the search head and peer node requirements of each site, as determined by your search affinity and disaster recovery needs. At a minimum, you will need (replication factor + 2) instances.

__Configure the master node__  

You configure the key attributes for the entire cluster on the master node. Here is an example of a multisite configuration for a master node:

```
[general]
site = site1

[clustering]
mode = master
multisite = true
available_sites = site1,site2
site_replication_factor = origin:2,total:3
site_search_factor = origin:1,total:2
pass4SymmKey = whatever
cluster_label = cluster1
```
This example specifies that:

* the instance is located on site1.
* the instance is a cluster master node.
* the cluster is multisite.
* the cluster consists of two sites: site1 and site2.
* the cluster's replication factor is the default "origin:2,total:3".
* the cluster's search factor is "origin:1,total:2".
* the cluster's security key is "whatever".
* the cluster label is "cluster1."
 
 Note the following:  
* You specify the site attribute in the [general] stanza.
* You specify all other multisite attributes in the [clustering] stanza.
* You can locate the master on any site in the cluster, but each cluster has only one master.
* You must set multisite=true.
* You must list all cluster sites in the available_sites attribute.
* You must set a site_replication_factor and a site_search_factor. For details, see Configure the site replication factor and Configure the site search factor.
* The pass4SymmKey attribute, which sets the security key, must be the same across all cluster nodes. See Configure the indexer cluster with server.conf for details.
* The cluster label is optional. It is useful for identifying the cluster in the monitoring console. 

[top](#toc)


## Create a Cluster Master <a name="create_cluster_master"></a>

If you are going to create a indexer cluster, you must also create a 'master node' (Splunk nomenclature) - aka a Cluster Master - to manage the cluster.  

__To configure a Splunk Enterprise instance as the Cluster Master / master node:__

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable indexer clustering.
4. Select Master node and click Next.
5. There are a few fields to fill out:

* Replication Factor. The replication factor determines how many copies of data the cluster maintains. The default is 3. Be sure to choose the right replication factor now. It is inadvisable to increase the replication factor later, after the cluster contains significant amounts of data.  

* Search Factor. The search factor determines how many immediately searchable copies of data the cluster maintains. The default is 2. Be sure to choose the right search factor now. It is inadvisable to increase the search factor later, once the cluster has significant amounts of data.  

* Security Key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. The value that you set here must be the same that you subsequently set on the peers and search heads as well.  

* Cluster Label. You can label the cluster here. The label is useful for identifying the cluster in the monitoring console. See Set cluster labels in Monitoring Splunk Enterprise.

6. Click Enable master node. The message appears, "You must restart Splunk for the master node to become active. You can restart Splunk from Server Controls."

7. Create an outputs.conf file in ```/opt/splunk/etc/system/local/``` on the master node that configures it for load-balanced forwarding of its internal log files across the set of peer nodes. You must also turn off indexing on the master, so that the master does not both retain the data locally as well as forward it to the peers.

	Here is an example outputs.conf file that accomplishes these requirements:
```
# Turn off indexing on the master
[indexAndForward]
index = false
 
[tcpout]
defaultGroup = my_peers_nodes 
forwardedindex.filter.disable = true  
indexAndForward = false 
 
[tcpout:my_peers_nodes]
server=10.10.10.1:9997,10.10.10.2:9997,10.10.10.3:9997
autoLB = true
```

8. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.

	Important: When the master starts up for the first time, it will block indexing on the peers until you enable and restart the full replication factor number of peers. Do not restart the master while it is waiting for the peers to join the cluster. If you do, you will need to restart the peers a second time.

9. After the restart, log back into the master and return to the Clustering page in Splunk Web (Settings > Indexer Clustering>). You will see the master clustering dashboard. 

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
__Restarting a Cluster Master and/or peers__

When restarting cluster peers, you should use Splunk Web or one of the cluster-aware CLI commands, such as splunk offline or splunk rolling-restart. Do not use splunk restart.

If, for any reason, you do need to restart both the master and the peer nodes:

* Restart the master node using __'splunk restart'__  
* Restart the peers as a group, using __'splunk rolling-restart cluster-peers'__

You might occasionally have need to restart a single peer; for example, if you change certain configurations on only that peer. There are two ways that you can safely restart a single peer:

* Use Splunk Web (Settings>Server Controls).
* Run the command __'splunk offline'__, followed by __'splunk start'__.

When you use Splunk Web or the splunk offline/splunk start commands to restart a peer, the master waits 60 seconds (by default) before assuming that the peer has gone down for good. This allows sufficient time for the peer to come back on-line and prevents the cluster from performing unnecessary remedial activities.

### Managing Configurations Across Peers <a name="managing_peer_configs"></a>


```splunk apply cluster-bundle```


#### Configuring Indexes on a Cluster <a name="configuring_indexes_cluster"></a>

If you are using a index cluster, you must configure custom indexes in the indexes.conf on the Cluster Master in the application-specific directory under the ```/opt/splunk/etc/master-apps/...``` directory.

The ```maxTotalDataSizeMB``` and ```frozenTimePeriodInSecs``` attributes in indexes.conf determine when buckets roll from cold to frozen. 

If you set the coldToFrozenDir attribute in indexes.conf, the indexer will automatically copy frozen buckets to the specified location before erasing the data from the index.

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

__Cluster Master failure__

If a master node goes down, peer nodes can continue to index and replicate data, and the search head can continue to search across the data, for some period of time. Problems eventually will arise, however, particularly if one of the peers goes down. There is no way to recover from peer loss without the master, and the search head will then be searching across an incomplete set of data. Generally speaking, the cluster continues as best it can without the master, but the system is in an inconsistent state and results cannot be guaranteed.

[top](#toc)

## Create a Splunk Search Head <a name="create_search_head"></a>

__Distributed search__

To set up a simple distributed search topology, consisting of a single dedicated search head and several search peers, perform these steps:

1. Designate a Splunk Enterprise instance as the search head. Since distributed search is enabled automatically on every full Splunk Enterprise instance, you do not actually perform any action in this step, aside from choosing the instance that you want to be your search head.

2. Establish connections from the search head to all the search peers that you want it to search across. This is the key step in the procedure:

__See Add search peers to the search head.__


3. Forward the search head's internal data to the search peers (indexers):

* Create an outputs.conf file on the search head that configures the search head for load-balanced forwarding across the set of search peers (indexers).  

* Turn off indexing on the search head, so that the search head does not both retain the data locally as well as forward it to the search peers.

Here is an example outputs.conf file:
```
# Turn off indexing on the search head
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

4. Log in to the search head and perform a search that runs across all the search peers, such as a search for *. Examine the 'splunk_server' field in the results to verify that all the search peers are listed in that field.

To increase the search management capacity, deploy multiple search heads as members of a search head cluster.

### Distributed Search <a name="distributed_search"></a> 

There are two basic options for deploying a distributed search environment:

* One or more independent search heads to search across the search peers.

* Multiple search heads in a search head cluster.
	
A search head cluster is a group of search heads that work together to provide scalability and high availability. The search heads in the cluster share resources, configurations, and jobs. This offers a way to scale a deployment transparently to users. 

The search heads in a cluster are, for most purposes, interchangeable. All search heads have access to the same set of search peers. They can also run or access the same searches, dashboards, knowledge objects, and so on.

In each case, the search heads perform only the search management and presentation functions. They connect to search peers (indexers) that index data and search across the indexed data.

When initiating a distributed search, the search head replicates and distributes its knowledge objects to its search peers (indexers) so that they can properly execute queries on its behalf. Knowledge objects include saved searches, event types, and other entities used in searching across indexes; this set of knowledge objects is called the knowledge bundle.

Bundles typically contain a subset of files (configuration files and assets) from ```$SPLUNK_HOME/etc/system```, ```$SPLUNK_HOME/etc/apps```, and ```$SPLUNK_HOME/etc/users```. 

The process of distributing knowledge bundles means that peers by default receive nearly the entire contents of the search head's apps. If an app contains large binaries that do not need to be shared with the peers, you can eliminate them from the bundle and thus reduce the bundle size.

On the search head, the knowledge bundles resides under the ```$SPLUNK_HOME/var/run``` directory. The bundles have the extension .bundle for full bundles or .delta for delta bundles. They are tar files, so you can run tar tvf against them to see the contents.

The knowledge bundle gets distributed to the ```$SPLUNK_HOME/var/run/searchpeers``` directory on each search peer. 

On a search head cluster, you can view replication status from the search head cluster captain:  
Settings > Distributed Search > Search Peers  

[top](#toc)


## Create a Deployment Server

Note: You can use deployment server to distribute updates to search heads in indexer clusters, as long as they are standalone search heads. You cannot use the deployment server to distribute updates to members of a search head cluster - you must use a Deployer.  

[top](#toc)

## Create a License Server <a name="create_lic_server"></a>

#### Licenses for distributed search <a name="search_licenses"></a>

Each instance in a distributed search deployment must have access to a license pool. This is true for both search heads and search peers. See Licenses for search heads in Admin Manual.

[top](#toc)

## Install a Universal Forwarder <a name="install_uf"></a>

Forward data to an indexer
To forward remote data to an indexer, you use forwarders, which are Splunk Enterprise instances that receive data inputs and then consolidate and send the data to a Splunk Enterprise indexer.  

Forwarders come in two flavors:

* __Universal forwarders__. These maintain a small footprint on their host machine. They perform minimal processing on the incoming data streams before forwarding them on to an indexer, also known as the receiver.

* __Heavy forwarders__. These retain most of the functionality of a full Splunk Enterprise instance. They can parse data before forwarding it to the receiving indexer, store indexed data locally, and forward the parsed data to a receiver for final indexing on that machine as well.

Both types of forwarders tag data with metadata such as host, source, and source type, before forwarding it on to the indexer.  

Forwarders allow you to use resources efficiently when processing large quantities or disparate types of data coming from remote sources. They also offer capabilities for load balancing, data filtering, and routing.  

For most types of cluster deployments, you should enable indexer acknowledgment for the forwarders sending data to the peer. 

You can simplify the process of connecting forwarders to peer nodes by using the indexer discovery feature.

Add data inputs to the search peers. You add inputs in the same way as for any indexer, either directly on the indexer or through forwarders connecting to the indexer(s).
(what need to be done on each indexer - inputs.conf?)

[top](#toc)

## Splunk Common Network Ports <a name="network_ports"></a>

![Splunk Common Network Ports](/images/369-splunk-common-network-ports-ver1.5.jpg)  

The image above was obtained from <a href="http://jordan2000.com/splunk-common-network-ports/" target="_blank">Job Jordan's Blog: http://jordan2000.com/splunk-common-network-ports</a>

Other images in this document were obtained and/or modified from images copied from Splunk documentation. Splunk is not responsible for any inaccuracies introduced by my modifications.

[top](#toc)

> Written with [StackEdit](https://stackedit.io/).