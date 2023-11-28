---
layout: page
title: Self Checkout Analytics
description: Automated Self Checkout Analytics and Fault Detection for NCR Voyix Clients
img: assets/img/project1_img.png
importance: 1
category: 
related_publications: 
---

This is a description of my internship project at NCR (now [NCR Voyix](https://www.ncr.com/)) in summer 2023. NCR Voyix is the world's largest provider of self-checkout machines, software, and managed services for these devices - mostly around maintenance and repair. I was a part of the Data Engineering team which is responsible for building and upgrading the data collection and analysis tooling for machines deployed at client locations.

A common problem for self-checkout machines is that they have diverse failure modes. A failed scan could a real failure of the scanner or weighing scale hardware but more often, it is incorrect use by the customer in the form of an improper scan. Another common failure mode is improper integration of inventory management software and the payments software which leads to "missing" items. Often, employees block the scanner with an object to prevent customers from using it. These are just a few examples of the many failure modes that self-checkout machines can have. My project was to detect these failure modes and build an analytics system to automatically collect scanner data and build early warning systems for these failure modes. The idea is that early warning allows NCR to plan their maintenance schedules better saving operating costs for the company and capital efficiency for the client.

I started by studying the usage logs from eight scanners from Walmart and Aldi across three countries - the US, UK, and Germany. This part was somewhat manual (I was running SQL queries directly from Azure Data Studio) as the failure modes were not fully known and noticing bad usage patterns is more of an art. I regularly interviewed team members responsible for management of different failure modes and finalised 16 distinct patterns with my manager and mentor. At this point, the real engineering work started. My manager decided that 600 machines across two continents would be enough of a challenge for me from a data engineering and distributed systems perspective to make this a learning experience but not overwhelm me. 

I was in charge of systems design to final testing. Two questions guided my system design choices: scalability and cost. Analytics and data cloud are inherently expensive products and our target was a dashboard that was real-time within a 5-minute threshold. For data collection, I used the company's proprietary AFR systems deployed on NCR private cloud. The data returned by AFR is in XML format contains a lot of sensititve information. I wrote stress-tested Python scripts to process the XML to remove sensitive information. These scripts run on the private cloud due to legal requirements.

With the help of the database team, I designed a schema for a custom SQL Server on Azure database and implemented it as a managed instance - providing a blend of flexibility and cost efficacy. I wrote a Python script to process the XML data and insert it into the database. This script runs on Azure Functions and is accessible only with a custom API endpoint created with Azure API Management. Even if I say so myself, this is a clever design. Why? After conducting experiments with VM and Functions, I found the Azure Functions is cheaper and faster to scale. I could have used a blob storage trigger but that is slower and you must pay for the storage. An API endpoint costs almost nothing and the private cloud makes a direct request to this endpoint. The script updates the database. The database is the source of truth so we don't need AFR data anymore. A snapshot of the database is taken every 12 hours and stored in a blob storage container. This is the data source for the dashboard.

To make this powerful data accessible, I created Grafana dashboards for Walmart. The dashboard provided upto minute accurate information on operational performance at company level, store level, and unit level for the SCO devices. The dashboard also provided predictive analytics on what units might fail in the next month and sent a notification to the right company representative with suggested early maintenance. This used [Azure Communication Services](https://azure.microsoft.com/en-us/products/communication-services) to managed and send emails. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/grafana.png" title="Dashboard example" class="img-fluid rounded z-depth-1 img-center" %}
    </div>
</div>
<div class="caption">
    I cannot share actual dashboard pictures but it looked something like this.
</div>


At the end, this system was deployed to all 600 machines in the test batch processing 20 million log events, sending 18,000 dashboard updates, and sending 40 early maintenance notifications every day.
