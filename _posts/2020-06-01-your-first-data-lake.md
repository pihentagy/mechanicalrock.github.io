# Your First Lake

Data is ***so hot*** right now, no mistaking it. _Everyone_ wants it - even if they already have it.

If you don't already have a data lake, and are wondering what it is, or how you might begin creating yours, read on.

## What's a Data Lake? 

In most best practices, a data lake:
* is a single repository for all your data at rest
* has write once, read many properties (WORM)
* is hosted in the cloud

### Single Repository For Data At Rest

When we say a single repository, we mean a consistent approach to accessing data. As a user, I should have **consistent mechanisms** to:
* access all of the data 
* authenticate and authorize for the data lake
* have access rights to the data granted to me
* find out what data is available

It does not necessarily mean all data is stored in one cloud bucket. 

### Write Once Read Many 

The key concept to understand here, is that data lakes work best when **no objects or files are ever overwritten**. There are a few reasons why this is the case, some theoretical and some technical. 

The theoretical argument is, that by not overwriting any file, you have a ***full and complete history*** of your data that never changes. For this reason, most organisations stream change logs - typically table row level updates - to their data lakes as they happen, rather than copying snapshots of their databases to the data lake. Watch as we break this rule later on!

An example of a technical argument is that many organisations build **data warehouses** from their data lakes - a read heavy, **analytical repository**, that is used to generate data driven insights. Many of these systems rely on data not changing in the data lake. Even if they can detect a change in these files, how should they then process the rewritten file? Treat it as new data? Ignore it? Rewrite history? Mostly all impractical.

It is best to view a data lake as a chronological event store of the changes to your organisation's data over time.

### Hosted In The Cloud

It's good to start with the question, ***Can a Data Lake Be Hosted On Premise?*** - and the answer is, absolutely, *Yes*! But you will ultimately regret it - the only question is, how soon?

Here are some obvious benefits to a cloud hosted data lake
* Your data volume can scale endlessly
* All the best tools for working with data at scale are designed for cloud technologies and services
* You can feel confident in the availability, redundancy and restorability of your data

Sooner or later, you will run into one of these problems working with an on premise data lake:
* You are unable to scale your storage to accomodate your data
* You are unable to scale your bandwidth to allow computation on your data
* You are unable to scale the operational requirement of maintaining the data lake, stifling its natural growth

## Drilling a bore hole to water your Dry Lake Bed

This industry makes up a lot of silly metaphors to sell products, so I may as well continue the trend.

You may have seen articles discussing how to ***hydrate*** your data lake. What does this mean? It simply means, copying data to your data lake. Any of the following could apply, as long as the destination is your data lake bucket:
* Uploading CSV files manually
* A batch job to sync files from a File Share automatically
* Dumping the contents of a database or table in a format readable by data tools
* Continuously replicating the changes on a database
* Streaming real time data from IOT devices

The many different methods of getting data from somewhere, into your data lake bucket, are called ***ingestion***. Not hydration. Disregard what those greasy sales people tell you.


## Why we will be ingesting a database snapshot

In this post I want to focus on showing some tools for making use of your data lake, rather than ingestion, so I am going to fast track our ingestion using a ***database snapshot***. Here are some reasons why that may be a bad idea:
* Snapshots can be really big
* The data is instantly out of date
* The data available is only as it was at the time of the snapshot, you can not see a history of changes to it

And here are some reasons why that might be okay:
* You are copying the database of a recently de-commissioned system that has useful data, that you don't yet know how to make use of
* You want to experiment with some data tools. 
* It's more cost effective to snapshot a database at regular intervals than to continuously replicate it, and you can live with the drawbacks

## Let's get started

### ~~Hydrating~~ Ingesting
Ok so here I have a ready prepared Amazon RDS MySQL database - it has a bunch of data in it, but I don't know what exactly. If this was you, maybe this is your own Amazon RDS database, preferably your **non-production** one.

What I do know about **RDS** databases is that they take regular **snapshots** that you can use to restore the database to a given point in time if you need to. Even better, a recent feature has been released where you can [export a DB snapshot to an S3 Bucket](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ExportSnapshot.html). This writes a **parquet** file for each table in the database (across all schemas if your database has them).

 If you don't know what parquet is, just know two things about it - it contains a schema of the data it holds within the file (e.g. the columns it holds and what types they are), and it is columnar based, meaning it is optimised for you to obtain only some columns or equally all of the data from the file with great performance.

I've been through the Snapshot Export tutorial and set up the necessary roles and lo and behold, the S3 bucket of my choice happens to be my data lake bucket. Cleverly, I am allowed to specify a prefix into which the export files will be written, so I'm not treading on anyone's toes.

### We have data

The export was successful - let's navigate to the S3 browser to look at what was exported to the data lake bucket.

Ok - we can see a whole load of parquet files with different names. One for each table. But what's inside them? What columns might the table have? And what data types? To start realising some value from this data, we are going to have to understand the schema for it.

Enter **AWS Glue**. Glue does a number of things, but to summarise them all, it's service that helps people realise value from their data by allowing them to catalog it, load it, transform it  and rewrite somewhere else, on demand and at scale. The part we want right now is the catalog bit - we want Glue (Crawler) to look at our data and tell us what schemas it can find from the data. There are many, many tutorials and guides for using AWS Glue Crawlers to inspect your data and create a data catalog, so I will only show a couple of parts here.

Just a brief interlude to describe what a catalog is - it is metadata about your data. It describes how the data is structured (e.g. what tables you have, what columns those tables have and what types each column has). It does not store the data itself, though it usually will contain the location describing whereabouts the data can be found in your data lake bucket.

### What does the data look like?

Ok, so we've let loose a Glue Crawler on our data and it's told us it found a few tables. When a Glue Crawler crawls your source, it asks for a (catalog) database where it can write the tables and their schemas that it finds. This helps build up a logical view of how your data lake data is actually structured. Let's go have a look.

This makes sense - the tables it has found match the parquet filenames we saw exported to the bucket. As we click inside those tables in the Catalog, we can see how many columns a table has, what they are named and what the expected type is. That seemed quite easy. It's great to know the schema and the _shape_ of the data, but how can we make use of it?

### How do we work with the data?

You have the schema, you have the data on S3 in parquet format - what's next? The fastest and easiest way on AWS platform to take a look at your data at rest, is using **AWS Athena**. It is like a SQL workbench for files on S3, but it integrates with **Glue**, so you don't have to think about files anymore - it's like you are working with a database again! 

