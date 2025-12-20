---
title: Recycle Wizard + Serverless on AWS (Part 2)
categories:
- Frontend
- Web
tags:
- aws
- serveless
- API Gateway
- Lambda
- DynamoDB
- S3
- ios
- react native
- CloudSearch
- Rekognition
description: Add API Gatway, Lambda, DynamoDB, S3, Rekognition to the new version
  of Recycle Wizard
date: 2019-03-27
author_profile: true
classes: wide
---

The second part is search, which turned out to be trickier than I expected. The first version of the app requests the source data [here](https://secure.toronto.ca/cc_sr_v1/data/swm_waste_wizard_APR?limit=1000) when app loads, after using a few functions to clean and process the data, I used [fuse.js](https://fusejs.io/) to handle the search, especially for the fuzzy search. Although this approach of loading data first is slowing down the app loading process and what I want to have is to request an API with a query when needed instead of loading all data at the beginning.

The other reason I want to improve the way to handle the source data is because after I released the first version of the app, Open Data Toronto changed the format of the source data without keeping the old version, and it broke the code I wrote based on the old format. That's why I want to save the data to a place that I can have better control.

What came into my mind first is to put the data somewhere on AWS, such as RDS or DynamoDB, then I could use lambda function to run a query with the keyword from the app.

The source data looks like below:

```javascript
    [
    	{
    	"body": "&lt;ul&gt; \n &lt;li&gt;Place item in the &lt;strong&gt;Garbage Bin.&lt;/strong&gt;&lt;/li&gt; \n&lt;/ul&gt;",
    	"category": "Garbage",
    	"title": "Garbage (wrapping and tying)",
    	"keywords": "bread bag tag, milk bag tag, elastic band, rubber band, twist tie, rope, twine, string, hemp, ribbon, bow, burlap, staple, fastener, wire, florists wire, plastic tag, tape, duct tape, electrical tape, masking tape, scotch tape, painters tape, tape dispenser, chain, nylon, thread"
    	},
      ...
    ]
```

Because of the source data is already a JSON file and there aren't that many entries in the source data, I figured I could use DynamoDB to store them and run a query, but I encountered a couple of issues:

- DynamoDB [query operation](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html) finds items based on primary key values. You could use different attributes as query criteria by utilizing [global secondary values](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html), however it seems to be a overkill because what I want is to use a keyword to find the corresponding information.
- That means I need to use each keyword as the primary key. Since the source data puts all keywords in one string, I need to process the data so that each keyword  has the title, category, and body information.
- To upload the data to a dynamoDB table, you will have to follow certain syntax such as:

```javascript
        {
           "PutRequest":{
              "Item":{
                 "body":{
                    "S":"XXXX"
        					// s means this field is string
                 },
                 "category":{
                    "S":"XXXXX"
                 },
                 "title":{
                    "S":"xxxx"
                 },
                 "keyword":{
                    "S":"XXX"
                 }
              }
           }
        }

     And what's surprising is that the `BatchWriteItem` can only *comprise as many as 25 put or delete requests. Individual items to be written can be as large as 400 KB*.
```
The command I used to batch upload is `aws dynamodb batch-write-item --request-items file://[the file].json`

After I processed the source data with this snippet and upload to DynamoDB (select keyword as the primary key), I could use a keyword to run a query, However, the keyword has to be exact match, but it's a searching functionality, I definitely want things such as auto suggestion and fuzzy search. I can use a `contain` operator with `scan` ([since it is only available](https://stackoverflow.com/questions/43793888/how-to-make-search-using-contains-with-dynamodb) in `scan` not `query`) to find keyword that is part of a longer keyword, but scan goes through the whole table, it is not scalable and not good in terms of performance.

Those issues made me realized that it might be better to use other service who performs well in terms of auto suggestion and fuzzy search. ElasticSearch seems to be really good at searching in a large amount of data but it's a bit pricy for a small free app, then I found out [AWS CloudSearch](https://docs.aws.amazon.com/cloudsearch/latest/developerguide/getting-started.html), it's seems to be close to what I was looking for. And the good thing is that it can pull data directly from DynamoDB (or S3, local file). What you need to do is to create a search domain, upload or import a sample of the data so that CloudSearch can create index fields for the search. After the search domain has been initialized, don't forget to actually upload data to the search domain. One thing to note is that those steps will take a while depends on the size of the data. Once the search domain is up and running, you can run a test query within AWS or request the *Search Endpoint* with something like `[search domain url]/2013-01-01/search?q=basketball&return=category,title,f_body`. The other things I found is that when I tried to upload documents from DynamoDB, if you don't specify where the start point it, the importing seems to start with a random starting point, which will cause the imported documents is not complete.

One issue I found about importing data from Dynamo into CloudSearch is that it seems that sometime not all data is imported, only part of the data is imported. However I cannot reproduce the issue. Every time I import the same DynamoDB table, the *Searchable Documents* number is different.

Now onto the next problem, auto suggestion. CloudSearch has *Suggesters* where you can specify which field you would like to see auto suggestion, I added keyword (and it needs to reindex), but the result is just acceptable in my opinion

The last issue I found about CloudSearch is about text fields, in CloudSearch, Text and text-array fields are always searchable, Literal and literal-array fields can only be searched if they are search enabled in the domain's indexing options. In my case, I want the user to search only by keyword, not in body or title or category, I tried different index options, but found out that you can specify which field to search with `q.options`, for instance, `[search domain url]/2013-01-01/search?q=${query}&q.options={fields: ['keyword']}&return=keyword,category,title,f_body` will only search the query in the keyword filter, which will improve the relevancy of the search result. That's why I switched back to using CloudSearch for searching instead of using DynamoDB query.

Finally, to catch errors such as app crashing, I also added [react-native-sentry](https://github.com/getsentry/react-native-sentry) in the app.

There are a few things I'd like to learn and explore:

- To test serverless services locally and make the infrastructure as code, such as with [serverless framework](https://serverless.com/)
- Use [react native config](https://github.com/luggit/react-native-config) to manage the configuration for react native app
- Improve API Gateway, possibly add rate limiting and authentication
- See if it's possible to disable Sentry React Native in local env
