
# Install and Configure Splunk Enterprise

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

2. [Overview of a Clustered Splunk Environment](#clustered_overview)
	* [Indexer Clustering](#indexer_clustering)
		* [Managing an Indexer Cluster](#managing_indexer_cluster)
	* [Distributed Search](#distributed_search)
	*  

3. [Create a Splunk Search Head](#create_search_head)

4. [Create a Splunk Indexer](#create_indexer)

	[Index Time vs Search Time Processing](#index_vs_search)

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

__You are now ready to customize this instance of Splunk Enterprise for a specific function__

The following sections outline how to configure instances of Splunk Enterprise to perform specific functions (search head, indexer, etc.) in a distributed and/or clustered Splunk solution.   

It may be helpful to read the section '[Overview of a Clustered Splunk Environment](#clustered_overview)' below before progressing to specific component configuration.

If you are using a standalone instance of Splunk Enterprise you may want to apply some of the function-specific settings outlined in the following sections. Generic settings that may be applied to any Splunk solution will be indicated.

[top](#toc)

## Overview of a Clustered Splunk Solution <a name="clustered_overview"></a>

### Indexer Clustering <a name="indexer_clustering"></a> 

Splunk supports the concept of utilizing multiple indexers in an 'index cluster' to allow setting a 'replication factor' which creates multiple, redundant copies of rawdata files across multiple indexers so that if an indexer fails data is not lost, and 'search factor' wherein multiple copies of data indexes are distributed so that search performance is not affected if an indexer fails.

The image below depicts a simple clustered indexer solution:

![Clustered Indexer Solution](/images/Simplified_basic_cluster_60.png)  

There are two types of nodes in an indexer cluster:

*  A single Cluster Master to manage the cluster. This 'master node' is a specialized type of indexer that, while it doesn't participate in any data streaming, coordinates a range of activities involving the search peers and the search head, including coordinating which 'buckets' are replicated across the peer nodes for redundancy.

* Several 'peer nodes' that handle the indexing function for the cluster, indexing and maintaining multiple copies of the rawdata and index files, and running searches across the data.

#### Managing an Indexer Cluster <a name="managing_indexer_cluster"></a>

If you are using a index cluster, you must configure custom indexes in the indexes.conf on the Cluster Master in the application-specific directory under the ```/opt/splunk/etc/master-apps/...``` directory.


The ```maxTotalDataSizeMB``` and ```frozenTimePeriodInSecs``` attributes in indexes.conf determine when buckets roll from cold to frozen. 

If you set the coldToFrozenDir attribute in indexes.conf, the indexer will automatically copy frozen buckets to the specified location before erasing the data from the index.

The following is an example of typical custom index entries in an indexes.conf file:  
```
[<index_name>]
homePath   = volume:primary/index_name/db
coldPath   = volume:primary/index_name/colddb
thawedPath = $SPLUNK_DB/index_name/thaweddb
repFactor = auto
# index size = 1024 x MB
maxTotalDataSizeMB = 30720
# store data for 30 days before archiving
frozenTimePeriodInSecs = 2592000
coldToFrozenDir = "<path to frozen archive>"
```

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

## Create a Splunk Search Head <a name="create_search_head"></a>


__Distributed search__

To set up a simple distributed search topology, consisting of a single dedicated search head and several search peers, perform these steps:

1. Designate a Splunk Enterprise instance as the search head. Since distributed search is enabled automatically on every full Splunk Enterprise instance, you do not actually perform any action in this step, aside from choosing the instance that you want to be your search head.

2. Establish connections from the search head to all the search peers that you want it to search across. This is the key step in the procedure:

__See Add search peers to the search head.__


5. Forward the search head's internal data to the search peers. See Best practice: Forward search head data to the indexer layer.

Create an outputs.conf file on the search head that configures the search head for load-balanced forwarding across the set of search peers (indexers). You must also turn off indexing on the search head, so that the search head does not both retain the data locally as well as forward it to the search peers.

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

Log in to the search head and perform a search that runs across all the search peers, such as a search for *. Examine the 'splunk_server' field in the results to verify that all the search peers are listed in that field.

To increase the search management capacity, deploy multiple search heads as members of a search head cluster.

[top](#toc)

## Create a Splunk Indexer <a name="create_indexer"></a>

Add data inputs to the search peers. You add inputs in the same way as for any indexer, either directly on the search peer or through forwarders connecting to the search peer. See the Getting Data In manual for information on data inputs.

__About Indexers__

Splunk Enterprise stores all of the data it processes in indexes. An index is a collection of databases, which are subdirectories located in ```$SPLUNK_HOME/var/lib/splunk```. Indexes consist of two types of files: compressed rawdata files and index files. 

Splunk Enterprise comes with a number of preconfigured indexes, including:  

* main: This is the default Splunk Enterprise index. All processed data is stored here unless otherwise specified.
* _internal: Stores Splunk Enterprise internal logs and processing metrics.
* _audit: Contains events related to the file system change monitor, auditing, and all user search history.

#### Index Time vs Search Time Processing <a name="index_vs_search"></a>
Splunk Enterprise documentation contains references to the terms "index time" and "search time". These terms distinguish between the types of processing that occur during indexing, and the types that occur when a search is run.

It is better to perform most knowledge-building activities, such as field extraction, at search time. Index-time custom field extraction can degrade performance at both index time and search time.

Index-time processes take place between the point when the data is consumed and the point when it is written to disk.

The following processes occur during index time:

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

## Create a Deployment Server

[top](#toc)

## Install a Universal Forwarder

Forward data to an indexer
To forward remote data to an indexer, you use forwarders, which are Splunk Enterprise instances that receive data inputs and then consolidate and send the data to a Splunk Enterprise indexer.  

Forwarders come in two flavors:

* __Universal forwarders__. These maintain a small footprint on their host machine. They perform minimal processing on the incoming data streams before forwarding them on to an indexer, also known as the receiver.

* __Heavy forwarders__. These retain most of the functionality of a full Splunk Enterprise instance. They can parse data before forwarding it to the receiving indexer, store indexed data locally, and forward the parsed data to a receiver for final indexing on that machine as well.

Both types of forwarders tag data with metadata such as host, source, and source type, before forwarding it on to the indexer.  

Forwarders allow you to use resources efficiently when processing large quantities or disparate types of data coming from remote sources. They also offer capabilities for load balancing, data filtering, and routing.  



[top](#toc)

## Splunk Common Network Ports

![Splunk Common Network Ports](/images/369-splunk-common-network-ports-ver1.5.jpg)  

The image above was obtained from <a href="http://jordan2000.com/splunk-common-network-ports/" target="_blank">Job Jordan's Blog: http://jordan2000.com/splunk-common-network-ports</a>

Other images in this document were obtained and/or modified from images copied from Splunk documentation. Splunk is not responsible for any inaccuracies introduced by my modifications.

> Written with [StackEdit](https://stackedit.io/).