---
layout: post
title: "Processing Real-Time Data Streams with Azure Stream Analytics"
author: "Hersh Bhasin"
comments: true
categories: Azure-Stream-Analytics
published: true

---
# Introduction

Hello, my name is Hersh Bhasin and welcome to my course, "Processing Real-Time Data Streams with Azure Stream Analytics". I work as a Cloud Architect with... and have done quite a bit of work in helping transition workflows to a  stream-based architecture. 

# Objective

While streaming of data and stream processing is a vast topic and there is a whole world view that goes with it, this course is an introduction to the area.  I want to look at topics like:

1. What are the problems that streaming is trying to solve.
2. How does data streams differ from OLTP databases, key value stores
3. How can we listen on and extract data from an ever flowing stream of data.



#  Why Streaming?

## Our traditional storage approach locks up data  in tables & schemas 

Most of our typical web or mobile applications are build on a HTTP request/response paradigm. A web application makes a rest call to an api, that api gets the data from a database and returns it back to the application which then displays it in the UI.  Because the data that the application needs is stored away in well know table schemas in a database, a consuming application can efficiently query it with well know syntax . This works great when a known application talks to a known data store. However this also means that data gets locked away in tables and schemas

![](../assets/analytics_structured.PNG)



 *Pipeline Sprawl*

To extract data from multiple data stores, we need to build data pipelines and this can soon lead to a "pipeline sprawl".

![analytics_pipelinemess](../assets/analytics_pipelinemess.PNG)

Having a "well-known" data stores means that that data gets locked away in tables and  schemas, and this becomes a drawback when data has to be extracted from multiple stores, say by a offline batch process which needs to extract data from many databases and aggregate it for analytics. This  batch process would need to understand the specific schema of each of these databases,  unlock and open the door of each table as it were, and then perform transformation on the data. You can end up building hundreds of pipelines to extract data locked away and interwoven in these databases, caches and search indexes. This is inherently complicated and hard to manage.

# Data Streams



Instead of locking up our data in database tables,  we can think of data as an ever flowing river of events. Our applications and batch processes can subscribe to and listen on the events that flow in this river, and some events can even trigger other events. This emancipation of data is very powerful indeed.



![](../assets/analytics_unstructured.PNG)



#  What is Azure Stream Analytics?



"*Words are flowing out like endless rain into a paper cup*..." goes the famous Beatles song.  If our flowing data is the  rain of words, then our Stream Analytics job is  the tool that will collect some of the rain water and put it into a  cup.

  <img src="../assets/analytics_words.PNG" alt="Smiley face" height="600" width="425">  



Lets stretch the analogy of the Beatles song to its breaking point. 

* Applications cause events to occur. Maybe it is a sensor reporting the health of a device  every second. We can define an event as a fact about the world. Something happened in the world and the event is a record of it. It is a message with some information, maybe in the Jason format. 

* These  events fall like raindrops,  and the accumulated raindrops become a river,  and that river  could be an Event Hub, an IOT Hub or an Azure Blob Location.

* Now to this river, we take our paper cup to fill it with a subset of the data we are interested in. This cup could be a database or it could be an Azure blob store location, or an Azure data location, or it could be a real time Power BI dashboard.

* And the tool that we use to extract data from the river and fill our paper cup is an Azure Stream Analytics job. It bridges the input, say  an  event hub, to an output, say a sql server database, with a sql query. Then as the data come in real time into the river, the query runs against the unbounded stream of data as it arrives, and write out the  data of interest to the output. 

# Demo