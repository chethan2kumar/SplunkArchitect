# Creating and Using Summary Indexes

A summary index is a special index that stores the results of a scheduled report, when you enable summary indexing for the report. Summary indexing lets you run fast searches over large data sets by spreading out the cost of a computationally expensive report over time. To achieve this, the search that populates the summary index runs on a frequent, recurring basis and extracts the specific data that you require. You can then run fast and efficient searches against this small subset of data in the summary index.

[Creating a Summary Index](#create_index)
[Creating a Scheduled Search Report](#create_report)
[Using a Summary Index](#using_summary_index)

## Creating a Summary Index  <a name="create_index"></a>

A summary index is created just like any other Splunk index, but may contain or exclude specific settings to set an adequate size and retention period for the data to be stored.  

An example of the entries in an indexes.conf for a summary index is:

```
[summary_nge_exception_messages]
homePath = volume:primary/summary_nge_exception_messages/db
coldPath = volume:primary/summary_nge_exception_messages/colddb
thawedPath = $SPLUNK_DB/summary_nge_exception_messages/thaweddb
repFactor = auto
# DO NOT set a maxTotalDataSizeMB
# By default, max hot/warm bucket size is 1TB
# Index entries are fairly small for this summary
# DO NOT set a frozenTimePeriodInSecs
# This data should be retained for 6 years
# as a historical record
```

After configuring the index, apply the cluster bundle or restart Splunk as needed so that the new summary index will appear as a selection during scheduled report creation.

## Creating a Scheduled Search Report  <a name="create_report"></a>

You can create a Saved Search from a report that runs periodically to populate the summary index with time series data for later use. 

On a Search Head:  
Settings > Searches, reports, and alerts  
Select the App context to create the saved search in, if applicable  
New  

Complete the data fields per examples in the following images to create a report named 'Top 20 NGE Exception Messages':  

![Saved Search Report Setup 1](/images/saved_search_settings_1.png)  

The report is configured to collect data from the previous whole hour using the Earliest = -1h@h & Latest = -0h@h time specifiers.  

The search string used in the above report is:  
```
eventtype=fpp_gxp_services sourcetype=disney_nge_xbms nge_exception_message=* | rex field=nge_exception_message mode=sed "s/(#\d+])/#xxxx]/g" 
| rex field=nge_exception_message mode=sed "s/(\[.+])/[xxxx]/g"  
| rex field=nge_exception_message mode=sed "s/(Transaction timed out: deadline was .+$)/Transaction timed out: deadline was -date-/g"
| rex field=nge_exception_message mode=sed "s/(guest\/.*\/identifiers)/guest\/xxxx\/identifiers/g" 
| rex field=nge_exception_message mode=sed "s/(ID: .*$)/ID: xxxx/g" 
| rex field=nge_exception_message mode=sed "s/(For input string: ".*")/For input string "xxxx"/g" 
| rex field=nge_exception_message mode=sed "s/(End Date .* is prior to .*$)/End Date -date- is prior to -date-/g" 
| rex field=nge_exception_message mode=sed "s/(party for \d+ entitlement$)/party for xxxx entitlement/g" 
| rex field=nge_exception_message mode=sed "s/(found for \d+ for)/found for xxxx for/g" 
| top 20 nge_exception_message useother=true
| addinfo 
| eval start=strftime(info_min_time, "%Y-%m-%d %T") 
| eval end=strftime(info_max_time, "%Y-%m-%d %T") 
| eval _time = info_max_time
| fields start end count percent nge_exception_message 
```

__Notes on Search String Attributes__

The 'eventtype' in this example included an index and multiple host specifiers.  

* The 'rex' entries remove values within the exception messages that may be unique and replace them with generic 'xxxx' or '-data-' values so that the exception messages are reduced to just their 'types' so that accurate statistical analysis is possible.

* addinfo adds epoch time info fields to each event, including:  

	* info_min_time - the earliest time boundary for the search 
	* info_max_time  - the latest time boundary for the search
	* info_sid - the ID of the search that generated the event
	* info_search_time - the time the search was run

* The __eval _time = info_max_time__ creates a timestamp for the event from the info_max_time epoch value that reflects the *end* of the time range. The _time timestamp would otherwise reflect the time of the *start* of the time range, which can be misleading and doesn't lend itself well to time series analysis.  

* The designated fields, along with the rest of the addinfo fields, are added to the designated summary index as events.

This report is set to run at 5 minutes after each hour:  

![Saved Search Report Setup 2](/images/saved_search_settings_2.png)  

Near the bottom of the form, click to enable Summary indexing, select the (previously created) summary index, and if desired, any additional field(s) to help identify these events in the summary index:  

![Saved Search Report Setup 3](/images/saved_search_settings_3.png)  



Saving the scheduled report with summary indexing creates the following entries in ```/opt/splunk/etc/apps/wdpr_bog_dashboard/local/savedsearches.conf```; __notice the additional 'datasource' field__ which has been added so these events can be found / isolated from a summary index that may contain other event types:

```
[Top 20 NGE Exception Messages]
action.summary_index = 1
action.summary_index._name = summary_nge_exception_messages
action.summary_index.datasource = top_20_nge_exception_messages
alert.digest_mode = True
alert.suppress = 0
alert.track = 0
auto_summarize.dispatch.earliest_time = -1d@h
cron_schedule = 5 * * * *
description = Top 20 NGE Exception Messages by hour
dispatch.earliest_time = -1h@h
dispatch.latest_time = -0h@h
enableSched = 1
realtime_schedule = 0
search = eventtype=fpp_gxp_services sourcetype=disney_nge_xbms nge_exception_message=* earliest=-1h@h latest=-0h@h \
| rex field=nge_exception_message mode=sed "s/(#\d+])/#xxxx]/g" \
| rex field=nge_exception_message mode=sed "s/(\[.+])/[xxxx]/g"  \
| rex field=nge_exception_message mode=sed "s/(Transaction timed out: deadline was .+$)/Transaction timed out: deadline was -date-/g"\
| rex field=nge_exception_message mode=sed "s/(guest\/.*\/identifiers)/guest\/xxxx\/identifiers/g" \
| rex field=nge_exception_message mode=sed "s/(ID: .*$)/ID: xxxx/g" \
| rex field=nge_exception_message mode=sed "s/(For input string: ".*")/For input string "xxxx"/g" \
| rex field=nge_exception_message mode=sed "s/(End Date .* is prior to .*$)/End Date -date- is prior to -date-/g" \
| rex field=nge_exception_message mode=sed "s/(party for \d+ entitlement$)/party for xxxx entitlement/g" \
| rex field=nge_exception_message mode=sed "s/(found for \d+ for)/found for xxxx for/g" \
| top 20 nge_exception_message useother=true\
| addinfo \
| eval start=strftime(info_min_time, "%Y-%m-%d %T") \
| eval end=strftime(info_max_time, "%Y-%m-%d %T") \
| fields start end count percent nge_exception_message\
```

### Using a Summary Index <a name="using_summary_index"></a>

You search a summary index like you would any other Splunk index; you can leverage the 'datasource' field in this example to further isolate the desired events 
```
index=summary_nge_exception_messages datasource=top_20_nge_exception_messages | timechart span=1h limit=21 sum(count) by nge_exception_message
```

![Summary Index Search Example](/images/summary_index_search_example.png) 


> Written with [StackEdit](https://stackedit.io/).