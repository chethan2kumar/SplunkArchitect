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

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable indexer clustering.
4. Select Peer node and click Next.
5. There are a few fields to fill out:

* Master URI. Enter the master's URI, including its management port. For example: https://10.152.31.202:8089.
* Peer replication port. This is the port on which the peer receives replicated data streamed from the other peers. You can specify any available, unused port for this purpose. This port must be different from the management or receiving ports.
* Security key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. Set the same value here that you previously set on the master node.
6. Click Enable peer node.
7. Restart splunk
8. Repeat for all peer nodes

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