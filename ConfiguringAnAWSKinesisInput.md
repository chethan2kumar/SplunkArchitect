# Configuring an AWS Kinesis Input <a name="top"></a>

[Configuring Inputs](#configuring)
[Testing Inputs](#testing)


## Configuring Inputs
__This document provides instructions for configuring and testing new AWS Kinesis inputs for ingesting CloudWatch-based application and server logs via inputs on the Splunk Add-on for AWS application.__

1. On the applicable Heavy Forwarder, select 'Splunk Add-on for AWS' from the Apps drop-down.  You will see a list of any existing inputs:  

![Splunk Add-on for AWS](/images/Kinesis1.png)  

2. Click 'Create New Input', and select 'Kinesis' from the drop-down list:  

![New Kinesis Input](/images/SplunkAdd-OnForAWSNewInput.png)  

3. In the 'Regions' tab of the 'Add AWS Kinesis Input' form, enter or select the following information:  

	* Name - enter the name to be associated with the input, using the following convention:
	
		* the application short name  
		* three underscores (____) for spacing / legibility
		* the Business ID 
		* two colons (::)
		* the region in 'us-east-1' format
		* two colons (::)
		* the kinesis stream this input will be using
	* AWS Account - select the appropriate AWS role
	* Assume Role - do not configure
	* AWS Region - select the appropriate region    
	__Note:__ US East (Virginia) = us-east-1; US West (Oregon) = us-west-2
	
![Regions Tab](/images/Kinesis2.png)  


4. In the Templates tab enter or select the following:

* Stream Name - select the appropriate stream name (provided by the Cloud SE) for this input
* Initial Stream Position - select 'TRIM_HORIZON' 
* Encoding - leave at 'none'
* Record Format - select 'CloudWatchLogs'

![Templates Tab](/images/Kinesis3.png)

5. In the Settings tab enter or select the following:

	* Source Type: select 'aws:cloudwatchlogs:vpcflow' from the drop-down list
	* Index: manually enter the index the data is to be sent to  

![Settings Tab](/images/Kinesis4.png)

6. Click 'Create' - you will be returned to the initial application page view, and the new input should be listed. 
	* The new input will be immediately enabled and start ingesting data.

[Top](#top)

## Testing Inputs <a name="testing"></a>


[Top](#top)

> Written with [StackEdit](https://stackedit.io/).