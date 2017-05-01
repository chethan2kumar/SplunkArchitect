
# The Zen of Search  <a name="top"><a/>

[gentimes](#gentimes)  
[map](#map)  
[Search Command Quick Reference Table](#quickref)  
[Splunk Commands by Category](#category)  


#### __gentimes__ <a name="gentimes"></a>
Generates timestamp results starting with the exact time specified as start time. Each result describes an adjacent, non-overlapping time range as indicated by the increment value. This terminates when enough results are generated to pass the endtime value.

All hourly time ranges from December 1 to December 5 in 2014
```| gentimes start=12/1/14 end=12/5/14 increment=1h```  

All daily time ranges from 30 days ago until 27 days ago.
```| gentimes start=-30 end=-27```

* gentimes is useful in conjunction with the map command.   
* It does not work for future dates.  

[Top](#top)

#### __map__ <a name="map"></a>

Map is like a foreach iterator. It will take each "result" of a previous search, and perform the map search that many times with the specified map search. 



## __Search command quick reference table__ <a name="quickref"></a>

The table below lists all of the search commands in alphabetical order. There is a short description of the command and links to related commands. For the complete syntax, usage, and detailed examples, click the command name to display the specific topic for that command.

Some of these commands share functions. For a list of the functions with descriptions and examples, see Evaluation functions and Statistical and charting functions.

| Command | Description | Related commands | Notes / Examples |
| :------------------------ | :-------------------------------------------------- | :--------------------- | :---------------------------------------|
|abstract | Produces a summary of each search result.	 | highlight |
|accum	| Keeps a running total of the specified numeric field. | autoregress, delta, trendline, streamstats |
|addcoltotals | Computes an event that contains sum of all numeric fields for previous events. | addtotals, stats |
|addinfo | Add fields that contain common information about the current search. | search |
| addtotals | Computes the sum of all numeric fields for each result. | addcoltotals, stats | 
| analyzefields | Analyze numerical fields for their ability to predict another discrete field. | anomalousvalue | 
| anomalies | Computes an "unexpectedness" score for an event. | anomalousvalue, cluster, kmeans, outlier | 
| anomalousvalue | Finds and summarizes irregular, or uncommon, search results. | analyzefields, anomalies, cluster, kmeans, outlier | 
| anomalydetection | Identifies anomalous events by computing a probability for each event and then detecting unusually small probabilities. | analyzefields, anomalies, anomalousvalue, cluster, kmeans, outlier | 
| append | Appends subsearch results to current results. | appendcols, appendcsv, appendlookup, join, set | 
| appendcols | Appends the fields of the subsearch results to current results, first results to first result, second to second, etc. | append, appendcsv, join, set | 
| appendpipe | Appends the result of the subpipeline applied to the current result set to results. | append, appendcols, join, set | 
| arules | Finds association rules between field values. | associate, correlate | 
| associate | Identifies correlations between fields. | correlate, contingency | 
| audit | Returns audit trail information that is stored in the local audit index. |  | 
| autoregress | Sets up data for calculating the moving average. | accum, autoregress, delta, trendline, streamstats | 
| bin (bucket) | Puts continuous numerical values into discrete sets. | chart, timechart | 
| bucketdir | Replaces a field value with higher-level grouping, such as replacing filenames with directories. | cluster, dedup | 
| chart | Returns results in a tabular output for charting. See also, Statistical and charting functions. | bin,sichart, timechart | 
| cluster | Clusters similar events together. | anomalies, anomalousvalue, cluster, kmeans, outlier | 
| cofilter | Finds how many times field1 and field2 values occurred together. | associate, correlate | 
| collect | Puts search results into a summary index. | overlap | 
| concurrency | Uses a duration field to find the number of "concurrent" events for each event. | timechart | 
| contingency | Builds a contingency table for two fields. | associate, correlate | 
| convert | Converts field values into numerical values. | eval | 
| correlate | Calculates the correlation between different fields. | associate, contingency | 
| crawl | Crawls the filesystem for new sources to index. | input | 
| datamodel | Examine data model or data model dataset and search a data model dataset. | pivot | 
| dbinspect | Returns information about the specified index. |  | 
| dedup | Removes subsequent results that match a specified criteria. | uniq | 
| delete | Delete specific events or search results. |  | 
| delta | Computes the difference in field value between nearby results. | accum, autoregress, trendline, streamstats | 
| diff | Returns the difference between two search results. |  | 
| erex | Allows you to specify example or counter example values to automatically extract fields that have similar values. | extract, kvform, multikv, regex, rex, xmlkv | 
| eval | Calculates an expression and puts the value into a field. See also, Evaluation functions. | where | 
| eventcount | Returns the number of events in an index. | dbinspect | 
| eventstats | Adds summary statistics to all search results. | stats | 
| extract (kv) | Extracts field-value pairs from search results. | kvform, multikv, xmlkv, rex | extract kvdelim="=", pairdelim="," 
| fieldformat | Expresses how to render a field at output time without changing the underlying value. | eval, where | 
| fields | Removes fields from search results. |  |
| fieldsummary | Generates summary information for all or a subset of the fields. | analyzefields, anomalies, anomalousvalue, stats | 
| filldown | Replaces NULL values with the last non-NULL value. | fillnull | 
| fillnull | Replaces null values with a specified value. |  | 
| findtypes | Generates a list of suggested event types. | typer | 
| folderize | Creates a higher-level grouping, such as replacing filenames with directories. |  | 
| foreach | Run a templatized streaming subsearch for each field in a wildcarded field list. | eval | 
| format | Takes the results of a subsearch and formats them into a single result. |  | 
| from | Retrieves data from a dataset, such as a data model dataset, a CSV lookup, a KV Store lookup, a saved search, or a table dataset. |  | 
| gauge | Transforms results into a format suitable for display by the Gauge chart types. |  | 
| gentimes | Generates time-range results. | map | The gentimes command is a generating command and should be the first command in the search. __Generating commands use a leading pipe character__. Feed it a start datetime (MM/DD/YYYY[:HH:MM:SS])and optional end datetime and/or  increment. Example: All hourly time ranges from December 1 to December 5 in 2016: ```gentimes start=12/1/16 end=12/5/16 increment=1h``` |
| geom | Adds a field, named "geom", to each event. This field contains geographic data structures for polygon geometry in JSON and is used for the choropleth map visualization. | geomfilter | 
| geomfilter | Accepts two points that specify a bounding box for clipping a choropleth map. Points that fall outside of the bounding box are filtered out. | geom | 
| geostats | Generate statistics which are clustered into geographical bins to be rendered on a world map. | stats, xyseries | 
| head | Returns the first number n of specified results. | reverse, tail | 
| highlight | Highlights the specified terms. | iconify | 
| history | Returns a history of searches formatted as an events list or as a table. | search | 
| iconify | Displays a unique icon for each different value in the list of fields that you specify. | highlight | 
| input | Add or disable sources. |  | 
| inputcsv | Loads search results from the specified CSV file. | loadjob, outputcsv | 
| inputlookup | Loads search results from a specified static lookup table. | inputcsv, join, lookup, outputlookup | 
| iplocation | Extracts location information from IP addresses. |  | 
| join | Combine the results of a subsearch with the results of a main search. | appendcols, lookup, selfjoin | 
| kmeans | Performs k-means clustering on selected fields. | anomalies, anomalousvalue, cluster, outlier | 
| kvform | Extracts values from search results, using a form template. | extract, kvform, multikv, xmlkv, rex | 
| loadjob | Loads events or results of a previously completed search job. | inputcsv | 
| localize | Returns a list of the time ranges in which the search results were found. | map, transaction | 
| localop | Run subsequent commands, that is all commands following this, locally and not on remote peers. |  | 
| lookup | Explicitly invokes field value lookups. |  | 
| makecontinuous | Makes a field that is supposed to be the x-axis continuous (invoked by chart/timechart)	chart, timechart | 
| makemv | Change a specified field into a multivalued field during a search. | mvcombine, mvexpand, nomv | 
| makeresults | Creates a specified number of empty search results. |  | 
| map | A looping operator, performs a search over each search result. | gentimes | 
| metadata | Returns a list of source, sourcetypes, or hosts from a specified index or distributed search peer. | dbinspect | 
| metasearch | Retrieves event metadata from indexes based on terms in the logical expression. | metadata, search | 
| multikv | Extracts field-values from table-formatted events. |  | 
| multisearch | Run multiple streaming searches at the same time. | append, join | 
| mvcombine | Combines events in search results that have a single differing field value into one result with a multivalue field of the differing field. | mvexpand, makemv, nomv | 
| mvexpand | Expands the values of a multivalue field into separate events for each value of the multivalue field. | mvcombine, makemv, nomv | 
| nomv | Changes a specified multivalued field into a single-value field at search time. | makemv, mvcombine, mvexpand | 
| outlier | Removes outlying numerical values. | anomalies, anomalousvalue, cluster, kmeans | 
| outputcsv | Outputs search results to a specified CSV file. | inputcsv, outputtext | 
| outputlookup | Writes search results to the specified static lookup table. | inputlookup, lookup, outputcsv, outputlookup | 
| outputtext | Outputs the raw text field (_raw) of results into the _xml field. | outputtext | 
| overlap | Finds events in a summary index that overlap in time or have missed events. | collect | 
| pivot | Run pivot searches against a particular data model dataset. | dbinspect | 
| predict | Enables you to use time series algorithms to predict future values of fields. | x11 | 
| rangemap | Sets RANGE field to the name of the ranges that match. |  | 
| rare	Displays the least common values of a field. | sirare, stats, top | 
| regex | Removes results that do not match the specified regular expression. | rex, search | 
| relevancy | Calculates how well the event matches the query. |  | 
| reltime | Converts the difference between 'now' and '_time' to a human-readable value and adds adds this value to the field, 'reltime', in your search results. | convert | 
| rename | Renames a specified field; wildcards can be used to specify multiple fields. |  | 
| replace | Replaces values of specified fields with a specified new value. |  | 
| rest | Access a REST endpoint and display the returned entities as search results. |  | 
| return | Specify the values to return from a subsearch. | format, search | 
| reverse | Reverses the order of the results. | head, sort, tail | 
| rex | Specify a Perl regular expression named groups to extract fields while you search. | extract, kvform, multikv, xmlkv, regex | 
| rtorder | Buffers events from real-time search to emit them in ascending time order when possible. |  | 
| savedsearch | Returns the search results of a saved search. |  | 
| script, run | Runs an external Perl or Python script as part of your search. |  | 
| scrub | Anonymizes the search results. |  | 
| search | Searches indexes for matching events. |  | 
| searchtxn | Finds transaction events within specified search constraints. | transaction | 
| selfjoin | Joins results with itself. | join | 
| sendemail | Emails search results to a specified email address. |  | 
| set | Performs set operations (union, diff, intersect) on subsearches. | append, appendcols, join, diff | 
| setfields | Sets the field values for all results to a common value. | eval, fillnull, rename | 
| sichart | Summary indexing version of chart. | chart, sitimechart, timechart | 
| sirare | Summary indexing version of rare. | rare | 
| sistats | Summary indexing version of stats. | stats | 
| sitimechart | Summary indexing version of timechart. | chart, sichart, timechart | 
| sitop | Summary indexing version of top. | top | 
| sort | Sorts search results by the specified fields. | reverse | 
| spath | Provides a straightforward means for extracting fields from structured data formats, XML and JSON. | xpath | Example: pipe to spath, then pipe to rename Msg as _raw, then pipe to an extract (kv) or similar command, then pipe to table with a list of the desired fields 
| stats | Provides statistics, grouped optionally by fields. See also, Statistical and charting functions. | eventstats, top, rare | 
| strcat | Concatenates string values. |  | 
| streamstats | Adds summary statistics to all search results in a streaming manner. | eventstats, stats | 
| table | Creates a table using the specified fields. | fields | 
| tags | Annotates specified fields in your search results with tags. | eval | 
| tail | Returns the last number n of specified results. | head, reverse | 
| timechart | Create a time series chart and corresponding table of statistics. See also, Statistical and charting functions. | chart, bucket | 
| timewrap | Displays, or wraps, the output of the timechart command so that every timewrap-span range of time is a different series. | timechart | 
| top | Displays the most common values of a field. | rare, stats | 
| transaction | Groups search results into transactions. |  | 
| transpose | Reformats rows of search results as columns. |  | 
| trendline | Computes moving averages of fields. | timechart | 
| tscollect | Writes results into tsidx file(s) for later use by tstats command. | collect, stats, tstats | 
| tstats | Calculates statistics over tsidx files created with the | tscollect command. | stats, tscollect | 
| typeahead | Returns typeahead information on a specified prefix. |  | 
| typelearner | Generates suggested eventtypes. | typer | 
| typer | Calculates the eventtypes for the search results. | typelearner | 
| uniq | Removes any search that is an exact duplicate with a previous result. | dedup | 
| untable | Converts results from a tabular format to a format similar to stats output. Inverse of xyseries and maketable. |  | 
| where | Performs arbitrary filtering on your data. See also, Evaluations functions. | eval | 
| x11 | Enables you to determine the trend in your data by removing the seasonal pattern. | predict | 
| xmlkv | Extracts XML key-value pairs. | extract, kvform, multikv, rex | 
| xmlunescape | Unescapes XML. |  | 
| xpath | Redefines the XML path. |  | 
| xyseries | Converts results into a format suitable for graphing. | 

## Splunk Commands by Category <a name="category"></a>

The following tables list all the search commands, categorized by their usage. Some commands fit into more than one category based on the options that you specify.

The Splunk Documentation page has links for each command:
<a href="http://docs.splunk.com/Documentation/Splunk/6.5.3/SearchReference/Commandsbycategory" target="_blank">Splunk Search Reference - Commands by Category</a>

### Correlation
These commands can be used to build correlation searches.  

| Command | Description | 
| :------------------------ | :------------------------------------------------------------------------- |
| <a href="http://docs.splunk.com/Documentation/Splunk/6.5.3/SearchReference/Append" target="_blank">append</a> | Appends subsearch results to current results. | 
| appendcols | Appends the fields of the subsearch results to current results, first results to first result, second to second, etc. | 
| appendpipe | Appends the result of the subpipeline applied to the current result set to results. | 
| arules | Finds association rules between field values. | 
| associate | Identifies correlations between fields. | 
| contingency, counttable, ctable | Builds a contingency table for two fields. | 
| correlate | Calculates the correlation between different fields. | 
| diff | Returns the difference between two search results. | 
| join | Combines the of results from the main results pipeline with the results from a subsearch. | 
| lookup | Explicitly invokes field value lookups. | 
| selfjoin | Joins results with itself. | 
| set | Performs set operations (union, diff, intersect) on subsearches. | 
| stats | Provides statistics, grouped optionally by fields. See Statistical and charting functions. | 
| transaction | Groups search results into transactions. | 

### Data and indexes
These commands can be used to learn more about your data, add and delete data sources, or manage the data in your summary indexes.

#### View data
These commands return information about the data you have in your indexes. They do not modify your data or indexes in any way.  

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| audit | Returns audit trail information that is stored in the local audit index. | 
| datamodel | Return information about a data model or data model object. | 
| dbinspect | Returns information about the specified index. | 
| eventcount | Returns the number of events in an index. | 
| metadata | Returns a list of source, sourcetypes, or hosts from a specified index or distributed search peer. | 
| typeahead | Returns typeahead information on a specified prefix. | 

#### Manage data
These are some commands you can use to add data sources to or delete specific data from your indexes. | 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| crawl | Crawls the filesystem for new sources to add to an index. | 
| delete | Delete specific events or search results. | 
| input | Add or disable sources. | 
#### Manage summary indexes
These commands are used to create and manage your summary indexes. | 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| collect, stash | Puts search results into a summary index. | 
| overlap | Finds events in a summary index that overlap in time or have missed events. | 
| sichart | Summary indexing version of chart. Computes the necessary information for you to later run a chart search on the summary index. | 
| sirare | Summary indexing version of rare. Computes the necessary information for you to later run a rare search on the summary index. | 
| sistats | Summary indexing version of stats. Computes the necessary information for you to later run a stats search on the summary index. | 
| sitimechart | Summary indexing version of timechart. Computes the necessary information for you to later run a timechart search on the summary index. | 
| sitop | Summary indexing version of top. Computes the necessary information for you to later run a top search on the summary index. | 

### Fields 
These are commands you can use to add, extract, and modify fields or field values. The most useful command for manipulating fields is eval and its statistical and charting functions. | 

#### Add fields
Use these commands to add new fields.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| accum | Keeps a running total of the specified numeric field. | 
| addinfo | Add fields that contain common information about the current search. | 
| addtotals | Computes the sum of all numeric fields for each result. | 
| delta | Computes the difference in field value between nearby results. | 
| eval | Calculates an expression and puts the value into a field. See also, evaluation functions. | 
| iplocation | Adds location information, such as city, country, latitude, longitude, and so on, based on IP addresses. | 
| lookup | For configured lookup tables, explicitly invokes the field value lookup and adds fields from the lookup table to the events. | 
| multikv | Extracts field-values from table-formatted events. | 
| rangemap | Sets RANGE field to the name of the ranges that match. | 
| relevancy | Adds a relevancy field, which indicates how well the event matches the query. | 
| strcat | Concatenates string values and saves the result to a specified field. |

#### Extract fields
These commands provide different ways to extract new fields from search results.

| Command | Description | 
| :------------------------ | :------------------------------------------------------------------------- |
| erex | Allows you to specify example or counter example values to automatically extract fields that have similar values. | 
| extract, kv | Extracts field-value pairs from search results. | 
| kvform | Extracts values from search results, using a form template. | 
| rex | Specify a Perl regular expression named groups to extract fields while you search. | 
| spath | Provides a straightforward means for extracting fields from structured data formats, XML and JSON. | 
| xmlkv | Extracts XML key-value pairs. | 

#### Modify fields and field values
Use these commands to modify fields or their values.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| convert | Converts field values into numerical values. | 
| filldown | Replaces NULL values with the last non-NULL value. | 
| fillnull | Replaces null values with a specified value. | 
| makemv | Change a specified field into a multivalue field during a search. | 
| nomv | Changes a specified multivalue field into a single-value field at search time. | 
| reltime | Converts the difference between 'now' and '_time' to a human-readable value and adds adds this value to the field, 'reltime', in your search results. | 
| rename | Renames a specified field. Use wildcards to specify multiple fields. | 
| replace | Replaces values of specified fields with a specified new value. |

### Find anomalies
These commands are used to find anomalies in your data. Either search for uncommon or outlying events and fields or cluster similar events together.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| analyzefields, af | Analyze numerical fields for their ability to predict another discrete field. | 
| anomalies | Computes an "unexpectedness" score for an event. | 
| anomalousvalue | Finds and summarizes irregular, or uncommon, search results. | 
| anomalydetection | Identifies anomalous events by computing a probability for each event and then detecting unusually small probabilities. | 
| cluster | Clusters similar events together. | 
| kmeans | Performs k-means clustering on selected fields. | 
| outlier | Removes outlying numerical values. | 
| rare | Displays the least common values of a field. |
 
### Geographic and location
These commands add geographical information to your search results.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| iplocation | Returns location information, such as city, country, latitude, longitude, and so on, based on IP addresses. | 
| geom | Adds a field, named "geom", to each event. This field contains geographic data structures for polygon geometry in JSON and is used for choropleth map visualization. This command requires an external lookup with external_type=geo to be installed. | 
| geomfilter | Accepts two points that specify a bounding box for clipping choropleth maps. Points that fall outside of the bounding box are filtered out. | 
| geostats | Generate statistics which are clustered into geographical bins to be rendered on a world map. | 

### Prediction and trending
These commands predict future values and calculate trendlines that can be used to create visualizations. 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| predict | Enables you to use time series algorithms to predict future values of fields. | 
| trendline | Computes moving averages of fields. | 
| x11 | Enables you to determine the trend in your data by removing the seasonal pattern. | 

### Reports 
These commands are used to build transforming searches. These commands return statistical data tables that are required for charts and other kinds of data visualizations. | 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| addtotals | Computes the sum of all numeric fields for each result. | 
| autoregress | Prepares your events for calculating the autoregression, or moving average, based on a field that you specify. | 
| bin, discretize | Puts continuous numerical values into discrete sets. | 
| chart | Returns results in a tabular output for charting. See also, Statistical and charting functions. | 
| contingency, counttable, ctable | Builds a contingency table for two fields. | 
| correlate | Calculates the correlation between different fields. | 
| eventcount | Returns the number of events in an index. | 
| eventstats | Adds summary statistics to all search results. | 
| gauge | Transforms results into a format suitable for display by the Gauge chart types. | 
| makecontinuous | Makes a field that is supposed to be the x-axis continuous (invoked by chart/timechart) | 
| outlier | Removes outlying numerical values. | 
| rare | Displays the least common values of a field. | 
| stats | Provides statistics, grouped optionally by fields. See also, Statistical and charting functions. | 
| streamstats | Adds summary statistics to all search results in a streaming manner. | 
| timechart | Create a time series chart and corresponding table of statistics. See also, Statistical and charting functions. | 
| top | Displays the most common values of a field. | 
| trendline | Computes moving averages of fields. | 
| tstats | Performs statistical queries on indexed fields in tsidx files. | 
| untable | Converts results from a tabular format to a format similar to stats output. Inverse of xyseries and maketable. | 
| xyseries | Converts results into a format suitable for graphing. | 

### Results 
These commands can be used to manage search results. For example, you can append one set of results with another, filter more events from the results, reformat the results, and so on. 

#### Alerting
Use this command to email the results of a search. 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| sendemail | Emails search results, either inline or as an attachment, to one or more specified email addresses.

#### Appending
Use these commands to append one set of results with another set or to itself.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| append | Appends subsearch results to current results. | 
| appendcols | Appends the fields of the subsearch results to current results, first results to first result, second to second, and so on. | 
| join | SQL-like joining of results from the main results pipeline with the results from the subpipeline. | 
| selfjoin | Joins results with itself. | 

#### Filtering
Use these commands to remove more events or fields from your current results.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| dedup | Removes subsequent results that match a specified criteria. | 
| fields | Removes fields from search results. | 
| from | Retrieves data from a dataset, such as a data model dataset, a CSV lookup, a KV Store lookup, a saved search, or a table dataset. | 
| mvcombine | Combines events in search results that have a single differing field value into one result with a multivalue field of the differing field. | 
| regex | Removes results that do not match the specified regular expression. | 
| searchtxn | Finds transaction events within specified search constraints. | 
| table | Creates a table using the specified fields. | 
| uniq | Removes any search that is an exact duplicate with a previous result. | 
| where | Performs arbitrary filtering on your data. See also, Evaluation functions. | 

#### Formatting
Use these commands to reformat your current results. 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- | 
| untable | Converts results from a tabular format to a format similar to stats output. Inverse of xyseries and maketable. | 
| xyseries | Converts results into a format suitable for graphing. | 

#### Generating 
Use these commands to generate or return events.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| gentimes | Returns results that match a time-range. | 
| loadjob | Loads events or results of a previously completed search job. | 
| makeresults | Creates a specified number of empty search results. | 
| mvexpand | Expands the values of a multivalue field into separate events for each value of the multivalue field. | 
| savedsearch | Returns the search results of a saved search. | 
| search | Searches indexes for matching events. This command is implicit at the start of every search pipeline that does not begin with another generating command. | 

#### Grouping
Use these commands to group or classify the current results.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| cluster | Clusters similar events together. | 
| kmeans | Performs k-means clustering on selected fields. | 
| mvexpand | Expands the values of a multivalue field into separate events for each value of the multivalue field. | 
| transaction | Groups search results into transactions. | 
| typelearner | Generates suggested eventtypes. | 
| typer | Calculates the eventtypes for the search results. | 

#### Reordering 
Use these commands to change the order of the current search results.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| head | Returns the first number n of specified results. | 
| reverse | Reverses the order of the results. | 
| sort | Sorts search results by the specified fields. | 
| tail | Returns the last number N of specified results | 

#### Reading
Use these commands to read in results from external files or previous searches. 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| inputcsv | Loads search results from the specified CSV file. | 
| inputlookup | Loads search results from a specified static lookup table. | 
| loadjob | Loads events or results of a previously completed search job. | 

#### Writing 
Use these commands to define how to output current search results.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| outputcsv | Outputs search results to a specified CSV file. | 
| outputlookup | Writes search results to the specified static lookup table. | 
| outputtext | Ouputs the raw text field (_raw) of results into the _xml field. | 
| sendemail | Emails search results, either inline or as an attachment, to one or more specified email addresses. |
 
### Search 
| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| map | A looping operator, performs a search over each search result. | 
| search | Searches indexes for matching events. This command is implicit at the start of every search pipeline that does not begin with another generating command. | 
| sendemail | Emails search results, either inline or as an attachment, to one or more specified email addresses. | 
| localop | Run subsequent commands, that is all commands following this, locally and not on a remote peer. | 

### Subsearch
These are commands that you can use with subsearches. 

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| append | Appends subsearch results to current results. | 
| appendcols | Appends the fields of the subsearch results to current results, first results to first result, second to second, and so on. | 
| appendpipe | Appends the result of the subpipeline applied to the current result set to results. | 
| format | Takes the results of a subsearch and formats them into a single result. | 
| join | Combine the results of a subsearch with the results of a main search. | 
| return | Specify the values to return from a subsearch. | 
| set | Performs set operations (union, diff, intersect) on subsearches. | 

### Time 
Use these commands to search based on time ranges or add time information to your events.

| Command | Description |
| :------------------------ | :------------------------------------------------------------------------- |
| gentimes | Returns results that match a time-range. | 
| localize | Returns a list of the time ranges in which the search results were found. | 
| reltime | Converts the difference between 'now' and '_time' to a human-readable value and adds adds this value to the field, 'reltime', in your search results. |


> Written with [StackEdit](https://stackedit.io/) by James H. Baxter  