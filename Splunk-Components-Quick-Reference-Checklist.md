# Splunk Components Quick Reference Checklist

### Contents <a name="toc"></a>

1. [Installing Splunk Enterprise on Linux](#install_splunk)
2. [Uninstalling Splunk Enterprise on Linux](#uninstall_splunk)



## Installing Splunk Enterprise on Linux <a name="install_splunk"></a>

Access the Splunk website to obtain the latest Splunk Enterprise installation package:
<a href="https://www.splunk.com/en_us/download/sem.html?ac=ga_usa_brand_enterprise_exact_Mar17&_kk=splunk%2520enterprise&gclid=CIvWzN6Hk9MCFQsRgQodK_QARg" target="_blank">Download Splunk Enterprise RPM for Linux-x86_64</a>
__Create an account and/or login using your Splunk user ID & password.__

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





> Written with [StackEdit](https://stackedit.io/).