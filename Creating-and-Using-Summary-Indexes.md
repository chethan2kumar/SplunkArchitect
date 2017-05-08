# Creating and Using Summary Indexes <a name="top"></a>

A summary index is a designated Splunk index that stores the results of a scheduled report, when you enable summary indexing for the report. Summary indexing lets you run fast searches over large data sets by spreading out the cost of a computationally expensive report over time. To achieve this, the search that populates the summary index runs on a frequent, recurring basis and extracts the specific data that you require. You can then run fast and efficient searches against this small subset of data in the summary index.  

Summary indexes are also useful for storing historical time series data for statistical analysis, anomaly detection, and related machine learning efforts.  

1. [Creating a Summary Index](#create_index)  
2. [Creating a Scheduled Search Report](#create_report)  
3. [Using a Summary Index](#using_summary_index)  
4. [Additional Notes](#notes)
	* [Populating a Summary Index](#populating)
	* [Using the Delete command](#delete)
	* [Troubleshooting](#troubleshooting)
	

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

[Top](#top)
***

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

* The 'eventtype' in this example included an index and multiple host specifiers.  

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

Saving the scheduled report with summary indexing creates the following entries in ```/opt/splunk/etc/apps/wdpr_bog_dashboard/local/savedsearches.conf```; __notice the additional 'datasource' field__ which has been added so these events can be found / isolated from a summary index that may contain other event types.

Be aware that by default Splunk assigns the __source__ field to the name of the generating report - "Top 20 NGE Exception Messages" in this case - and the __sourcetype__ field to 'stash', which  is how splunk knows that data is already in splunk and the summary data will not be charged against the license.  

I have found that if you add an additional field called 'source' and give it a value, each event will contain two 'source' fields - one will have 'Top 20 NGE Exception Messages', for example, and the other will have the value assigned in the report generator - such as 'top_20_nge_exception_messages' - but performing a search based on the 2nd 'source' field does not work reliably (searching on the first 'source' field is reliable, however), so the best practice is to avoid using or reassigning either of these default fields and just use a new, unique field name such as 'datasource' to allow finding / isolating these specific events - or just search on the first source field. The only caveat to this approach is that if you artificially populate the summary index with a stand-alone search string (versus the scheduled search report) you either have to specify the the source field with the same verbiage ("Top 20 NGE Exception Messages") or rely on the other search field (datasource) to achieve reliable search results.

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
[Top](#top)
***

### Using a Summary Index <a name="using_summary_index"></a>

You search a summary index like you would any other Splunk index; you can leverage the 'datasource' field as is done in this example to further isolate the desired events in case there are additional data sources in the same index:  
```
index=summary_nge_exception_messages datasource=top_20_nge_exception_messages | timechart span=1h limit=0 sum(count) by nge_exception_message
```

![Summary Index Search Example](/images/summary_index_search_example.png) 

[Top](#top)

## Additional Notes <a name="notes"></a>

### Populating a Summary Index <a name="populating"></a>

After creating a scheduled search report, you will continue to add events to your summary index at the configured incremental times. You can also populate the summary index with previous / older data using a derivative of the following search string.  

The 'gentimes' command creates a set of 'starttime' and 'endtime' timestamps for the given timerange and increment.  

The 'map' command iterates over the piped set of timestamps and performs the given search using the $starttime$ and $endtime$ values for each time increment.  

The $starttime$ and $endtime$ values are also used to create start and end datetimes for these output fields, as well as to reset the _time value to the 'end' time as an output field as well. These fields, along with the nge_exception_message, count, and percentage fields, are then collected into the summary index.

Finally - note that because map's search command is enclosed in quotes, any quotes used with the search string itself must be delimited with a backslash ('\\').

__Remember to set the Splunk Web timerange to the same time range as the 'start' to 'end' period or you will get incomplete / no results.__

__This search takes a lot of time & resources to complete - use judiciously:__

```
| gentimes start=04/10/2017:00:00:00 end=04/24/2017:00:00:00 increment=1h@h 
| map maxsearches=720 search=" 
  search earliest=$starttime$ latest=$endtime$ eventtype=fpp_gxp_services sourcetype=disney_nge_xbms nge_exception_message=* 
| rex field=nge_exception_message mode=sed \"s/(#\d+])/#xxxx]/g\" 
| rex field=nge_exception_message mode=sed \"s/(\[.+])/[xxxx]/g\"  
| rex field=nge_exception_message mode=sed \"s/(Transaction timed out: deadline was .+$)/Transaction timed out: deadline was -date-/g\" 
| rex field=nge_exception_message mode=sed \"s/(guest\/.*\/identifiers)/guest\/xxxx\/identifiers/g\" 
| rex field=nge_exception_message mode=sed \"s/(ID: .*$)/ID: xxxx/g\" 
| rex field=nge_exception_message mode=sed \"s/(For input string: ".*")/For input string "xxxx"/g\" 
| rex field=nge_exception_message mode=sed \"s/(End Date .* is prior to .*$)/End Date -date- is prior to -date-/g\" 
| rex field=nge_exception_message mode=sed \"s/(party for \d+ entitlement$)/party for xxxx entitlement/g\" 
| rex field=nge_exception_message mode=sed \"s/(found for \d+ for)/found for xxxx for/g\" 
| top 20 nge_exception_message useother=t 
| eval start=strftime($starttime$, \"%Y-%m-%d %T\") 
| eval end=strftime(($endtime$ + 1), \"%Y-%m-%d %T\") 
| eval _time=$endtime$ + 1
| fields _time start end count percent nge_exception_message
| collect index=summary_nge_exception_messages source=populating_search  marker=\"datasource=top_20_nge_exception_messages\"
"

```
This avoids having to wait for some time period to have time series data available for creating time series / statistical analysis reports.  

[Top](#top)
***

### Using the Delete command <a name="delete"></a>

You can delete events from a summary index by creating a search that returns the events you want to delete, and then piping the results to the delete command. Be aware that the user ID you use to perform this operation will have to have the 'can delete' role assigned.  ___BE CAREFUL WITH THIS COMMAND!!!___  The delete function is permanent, and obviously, if misused you could erase more data than you wanted or even production indexes. You should be aware that the delete command can be used to delete an entire index, or all indexes if none is specified. A mistake here could be hazardous to your career.  

The example below illustrates how to delete events from a summary index:

1. Create a search specifying the desired summary index and a time range or other search filters

2. Run the search __without piping to the delete command__ to ensure that you're retrieving __*only*__ the events you want to delete: ```index=summary_nge_exception_messages earliest=-3d@d latest=-1d@d```

3. Add the | delete command to the end of the search string and run it again - for example:  ```index=summary_nge_exception_messages earliest=-3d@d latest=-1d@d | delete```

You should see something like the following for as many indexers as are storing buckets of data for this summary index:  

| splunk_server | index | deleted | errors |
| :--------------- | :---------------------------------------- | :----- | :--|
| fldcppsla2200	| ```__ALL__``` | 102 | 0 |
| fldcppsla2200	| summary_nge_exception_messages | 102 | 0 |
...  

[Top](#top)
***

### Troubleshooting <a name="troubleshooting"></a>

__Scheduled Report__  

You can verify that the scheduled report is running and validate the run parameters by searching the scheduler.log. Set the time range to include the last report run time and search for the report name:

```index=_internal source=*scheduler.log "Top 20 NGE Exception Messages" ```

You should see a savedsearch_name field with the name of your report, and a status=success field/value. An example of a status=success event is:  
```
04-26-2017 14:05:22.908 -0400 INFO  SavedSplunker - savedsearch_id="nobody;wdpr_bog_dashboard;Top 20 NGE Exception Messages", search_type="", user="baxtj018", app="wdpr_bog_dashboard", savedsearch_name="Top 20 NGE Exception Messages", priority=default, status=success, digest_mode=1, scheduled_time=1493229900, window_time=0, dispatch_time=1493229919, run_time=2.396, result_count=21, alert_actions="summary_index", sid="scheduler__baxtj018_d2Rwcl9ib2dfZGFzaGJvYXJk__RMD597c4f0496c342429_at_1493229900_53578_7965ABA8-8E64-445B-81A9-5573529732C7", suppressed=0, thread_id="AlertNotifierWorker-0"
host =	fldcvpsla9935 index =	_internal source =	/opt/splunk/var/log/splunk/scheduler.log sourcetype =	scheduler splunk_server =	fldcppsla2204
```

You may see previous events for the same report run where status=continued, status=delegated_remote, and status=delegated_remote_completion.

This search may help if you're sure the other report settings are correct and you just want to see if the search report ran successfully:

```index=_internal source=*scheduler.log "Top 20 NGE Exception Messages" | stats count by status```  

***
__Troubleshooting a Search String with Collect__  

You can troubleshoot the search string that is using the collect command by adding the 'spool=false' option to the collect command:

```| collect index=summary_nge_exception_messages spool=false source=populating_search  marker=\"datasource=top_20_nge_exception_messages\"```  

By default, __spool=true__ and Splunk saves any search results piped to the collect command into files named ```<unique alphanumeric>_events.stash_new``` in the ```/opt/splunk/var/spool/splunk/``` directory. This is a 'monitored' directory; any files placed here are ingested into the appropriate index, which is (should be) specified in the header of the file, and then deleted - so they don't stay there very long.  

Setting __spool=false__ will cause Splunk to save the search results into files named ```<unique alphanumeric>_events.stash``` in the ```/opt/splunk/var/run/splunk/``` directory. You can 'more' this file (if it exists) to see if the datetime and other fields are as you expected.  

The header and first two events in a typical file created by the collect command are as follows:  

```
***SPLUNK*** index=summary_nge_exception_messages source=populating_search
04/20/2017 19:00:00 -0400, info_min_time=1492660800.000, info_max_time=1493006400.000, info_search_time=1494272376.009, count=2475, e
nd="2017-04-20 19:00:00", nge_exception_message="Guest Offer Set contains at least one booked offer!", percent="64.085966", start="20
17-04-20 18:00:00", datasource=top_20_nge_exception_messages
04/20/2017 19:00:00 -0400, info_min_time=1492660800.000, info_max_time=1493006400.000, info_search_time=1494272376.009, count=185, en
d="2017-04-20 19:00:00", nge_exception_message="Xpass status is not booked.", percent="4.790264", start="2017-04-20 18:00:00", dataso
urce=top_20_nge_exception_messages
```  
Note again that a search using the map + collect command as exemplified earlier in this document may take a __really - long - time__ to finish if you're spanning longer time periods.  Splunk appears to give such searches a very low priority, and quite some time goes by before you start seeing files generated in the /spool directory and events start showing up in your summary index.  

It may be prudent to test a complex or long-running search using the spool=false option to ensure your search works properly before removing this option and allowing the 'real' search to run.  

[Top](#top)
***
> Written with [StackEdit](https://stackedit.io/) by James H. Baxter - last update 5/4/2017