# NOTE

This repository is NOT complete, or completely accurate. It was started - and later abandoned - during my journey towards becoming a certified Splunk architect. I recently noticed that some folks have been forking and/or starring this - and I highly recommend NOT using this as a reference.  

I DO recommend my book: Splunk 7.x Quick Start Guide available from Packt Publising and Amazon, which *is* a complete and accurate reference - I use it myself on a very regular basis to fill in the gaps for all the stuff you can't remember if you don't do it every day:  

https://www.amazon.com/Splunk-7-x-Quick-Start-Guide-ebook/dp/B07L1MQF4V  

My apologies - I should have been a more responsible repository owner.  

Also: As time permits, I will be migrating the usable data from this repository to my Machine Data Insights repository located here:  

https://github.com/machinedatainsights  

___

# SplunkArchitect

This is a repository for a set of markdown files initially created as a study and reference guide for passing the Splunk Architect certification lab.

A secondary purpose is a set of notes for building a clustered Splunk environment for both on premise and AWS environments.

From the <a href="http://www.splunk.com/view/SP-CAAAH9R" target="_blank">Splunk Architect Certification Lab</a> link:

## Splunk Architect Certification Lab

This 24-hour practical exam is designed to assess the skills and knowledge of Splunk Certified Architect candidates and is the final step toward certification. Each participant is given access to a specified number of Linux servers and a set of requirements. Participants then perform a mock deployment according to requirements which adhere to Splunk Deployment Methodology and best-practices.

### Lab Format

The lab is facilitated by a live instructor via virtual classroom. Participants are allowed 24 hours continuous access to the servers to complete the requirements. A live instructor is available for the first 4 hours for direct facilitation.  

### Prerequisites

* Using Splunk
* Searching and Reporting with Splunk
* Creating Splunk Knowledge Objects
* Splunk Administration
* Advanced Dashboards and Visualizations
* Architecting and Deploying Splunk

** 30 days hands-on Splunk experience following completion of above courses is recommend prior to attending the Certification Lab.

### Course Objectives

__Installation and Infrastructure__

[Install a search head, deployment server and indexers](https://github.com/packetiq/SplunkArchitect/blob/master/Install-and-Configure-Splunk-Enterprise-Components.md)  
[Perform a scripted installation of universal forwarders](https://github.com/packetiq/SplunkArchitect/blob/master/Install-and-Configure-Splunk-Enterprise-Components.md#install_uf)  

__Configuration, Collection, and Comprehension__

[Deploy all specified configurations via deployment server](https://github.com/packetiq/SplunkArchitect/blob/master/Install-and-Configure-Splunk-Enterprise-Components.md#create_deployment_svr)  
[Gather data from forwarders and send to multiple indexes depending on use case](https://github.com/packetiq/SplunkArchitect/blob/master/Install-and-Configure-Splunk-Enterprise-Components.md#create_deployment_apps)  
Configure and confirm index-time knowledge  
Create search time field extractions  

__Searching and Reporting__

Create searches and dashboards for each required use case  



