# __Configuring an AWS Kinesis Input__ <a name="top"></a>

__This document provides instructions for configuring and testing new AWS Kinesis inputs used for delivering CloudWatch-based application and server logs to inputs on the Splunk Add-on for AWS application running on a Heavy Forwarder.__  

[Configuring Inputs](#configuring)  
[Testing Inputs](#testing)  
[Troubleshooting Inputs](#trouble)  
[Performance Metrics](#performance)  
***
## Configuring Inputs <a name="configuring"></a>

1. On the applicable indexer / index cluster, ensure the __index__ the new input data is going to go to has been configured.

2. On the applicable Heavy Forwarder, select 'Splunk Add-on for AWS' from the Apps drop-down.  You will see a list of any existing inputs:  

![Splunk Add-on for AWS](/images/Kinesis1.png)  

3. Click 'Create New Input', and select 'Kinesis' from the drop-down list:  

![New Kinesis Input](/images/SplunkAdd-OnForAWSNewInput.png)  

4. In the 'Regions' tab of the 'Add AWS Kinesis Input' form, enter or select the following information:  

	* Name - enter the name to be associated with the input, using the following convention:
	
		* the application short name  (provided by the app team or derived otherwise)
		* three underscores (____) for spacing / legibility
		* the Business ID (BAPP ID)
		* two colons (::)
		* the region in 'us-east-1' format
		* two colons (::)
		* the kinesis stream this input will be using
	* AWS Account - select the appropriate AWS role
	* Assume Role - do not configure
	* AWS Region - select the appropriate region    
	__Note:__ US East (N. Virginia) = us-east-1; US West (Oregon) = us-west-2
	
![Regions Tab](/images/Kinesis2.png)  


5. In the Templates tab enter or select the following:

* Stream Name - select the appropriate stream name (provided by the Cloud SE) for this input
* Initial Stream Position - select 'TRIM_HORIZON' (including older records) vs 'LATEST' (recent records only)
* Encoding - leave at 'none'
* Record Format - select 'CloudWatchLogs'

![Templates Tab](/images/Kinesis3.png)

6. In the Settings tab enter or select the following:

	* Source Type: select 'aws:cloudwatchlogs:vpcflow' from the drop-down list
	* Index: manually enter the index the data is to be sent to  

![Settings Tab](/images/Kinesis4.png)

7. Click 'Create' - you will be returned to the initial application page view, and the new input should be listed. 
	* The new input will be immediately enabled and start ingesting data.

[Top](#top)
***
## Testing Inputs <a name="testing"></a>

You can test for proper input operation by simply searching for events in the index you directed the input to:

```index=<TheRightIndex> | stats count by source```

Use asterisks and partical result strings to search for a specific source - in this case, AWS instance server logs:  

``` index=<TheRightIndex> source=*-i-* | stats count by source```

Be aware that when you initially create the input using the 'TRIM_HORIZON' option in the Templates tab, it will take some time for all the older events to be ingested before the more recent events will appear in a search. Select a timerange of 'All time' to see the older events immediately after creating the input.

[Top](#top)
***

## Troubleshooting Inputs <a name="trouble"></a>

Most initial problems with a new input will be caused by one of the following:

* Improper index name
* Missing index
* Improper kinesis stream configuration (get the Cloud SE to verify)
* The application isn't running, and therefore no logs are being created to be ingested  

To assist in troubleshooting more difficult input issues, you should be forwarding Splunk internal logs on the Heavy Forwarder to the indexers so that they can be searched:   

[Install-and-Configure-Splunk-Enterprise-Components#fwd_internal_data](./Install-and-Configure-Splunk-Enterprise-Components.md#fwd_internal_data)

Logs forwarded from the Heavy Forwarder(s) should appear in the _internal and other applicable Splunk logs on the indexer/cluster. You will need to utilize source and/or sourcetype filtering to isolate the events of interest from other Splunk logging events:  

__Searching for kinesis input events:__
index=_internal sourcetype=aws:cloudwatchlogs:log 

__Searching for throttling events:__
index=_internal Throttling

Note that you will see several events reflecting your search for 'Throttling'

For a complete discussion on troubleshooting the Splunk Add-on for AWS see the Splunk docs:    
<a href="http://docs.splunk.com/Documentation/AddOns/released/AWS/Troubleshooting" target="_blank">Troubleshoot the Splunk Add-on for AWS</a>

[Top](#top)
***

## Performance Metrics <a name="performance"></a>

__Throughput__

You can get metrics for how much throughput your Heavy Forwarder and Indexers are handling with this search string:  

```index=_internal sourcetype=splunkd source=*metrics.log group=per_host_thruput | stats avg(kbps) as avg_kbps max(kbps) as max_kbps avg(eps) as avg_eps max(eps) as max_eps avg(kb) as avg_kb max(kb) as max_kb avg(ev) as avg_events max(ev) as max_events by host```

A more granular search that can be run for multiple days:

```index=_internal sourcetype=splunkd source=*metrics.log group=per_host_thruput | bucket span=1m _time | stats max(kbps) as max_kbps max(eps) as max_eps sum(kb) as kb sum(ev) as ev by _time,host | bucket span=1h _time | stats max(max_kbps) as max_kbps max(max_eps) as max_eps max(kb) as max_kbpm max(ev) as max_epm sum(kb) as kb sum(ev) as ev by _time, host | bucket span=1d _time | stats max(max_kbps) as max_kbps max(max_eps) as max_eps  max(max_kbpm) as max_kbpm max(max_epm) as max_epm max(kb) as max_kbph max(ev) as max_eph sum(kb) as kb sum(ev) as ev by _time, host```

__Concurrent Searches__

This search will return the number of concurrent searches per hour for the last 30 days. Concurrent searches has nothing to do with Heavy Forwarder performance but if you're troubleshooting performance issues this may be worth taking a look at as well.  

```index=_internal source=*metrics.log group="search_concurrency" earliest=-1month@month latest=-0month@month | timechart span=1h sum(active_hist_searches) as concurrent_searches```

__Active Users__

```index=_internal source=*metrics.log group="search_concurrency" earliest=-1month@month latest=-0month@month | timechart sum(active_hist_searches) as concurrent_searches by user```

__Saved Searches__

```index="_internal" sourcetype="scheduler" ```

__NOTE:__ For saved searches, there are all sorts of fields to play with, including *savedsearch_name, status, app, run_time*, and more.

__Manual Searches__

```index="_internal" sourcetype="searches"```

See the [SplunkInternalIndexSourceSourcetypeList.xlsx](./SplunkInternalIndexSourceSourcetypeList.xlsx) document for a complete list of Splunk internal logs and their various source and sourcetypes which can be investigated for additional insights into Splunk operation and performance.  

[Top](#top)
***
> Written with [StackEdit](https://stackedit.io/) by James H. Baxter - last update 5-2-2017.