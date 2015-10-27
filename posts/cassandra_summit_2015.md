---
title: Notes on Cassandra Summit 2015
author: carlos
date: "2015-10-27"
description: Notes and highlights from Cassandra Summit 2015
tags: ["Cassandra", "NoSQL", "DataStax"]
categories: ["conferences", "talks"]
---

# Cassandra Summit 2015

![Cassandra Summit ](/assets/media/cassandra_summit_2015.png)

DataStax Cassandra Summit is becoming a GREAT conference!! With each new edition more and more people, talks and activities are adding themselves to the event to make it really awesome.

This year has been very special and intense for me, for several reasons.

*   First because of the location. This year, the Summit was held in Santa Clara, CA. Which meant an opportunity for me to know the area, including San Francisco and The Bay. I've really enjoyed walking and visiting the city's touristic attractions. San Francisco is a lovely city and the weather has been just amazing, but I wouldn't like to close this note without mentioning the reality of the streets. It is simply incredible the amount of homeless and people suffering mental diseases you can see just by walking the city. And it seems that the situation is getting worse as the housing prices increase. This article on the news particularly caught my attention: [Tech bus drivers forced to live in cars to make ends meet](http://www.sfchronicle.com/business/article/Tech-bus-drivers-forced-to-live-in-cars-to-make-6517928.php).

*   Second reason is because I have spoken there!! This has been, by far, the biggest conference I've had the pleasure to speak at. And I really enjoyed it!! The talk was actually a live coding demo (best kind of talks for sure, aren't they?) showing how I, being a Cassandra developer, not admin, tackled a production issue a couple of months ago. You can see the exact steps in [this previous blog post](http://engineering.mydrivesolutions.com/posts/cassandra_10_seconds_back/) and also in [this video](https://www.youtube.com/watch?v=nwscaRTmVCM) of this same talk in [the London Cassandra Meetup](http://www.meetup.com/Cassandra-London/events/224555591/).

*   Third reason is because I've been officially certified as an Apache Cassandra Developer by DataStax and O'Reilly Media: <https://twitter.com/calonso/status/646480545514258432>

*   And finally, because I've been awarded with one of [the DataStax MVPs of the year 2015](http://www.planetcassandra.org/mvps/)!! And, to be honest, I couldn't be more excited. I love Cassandra and I love contributing to OS projects and getting some recognition on it is always welcome. I'd like to congratulate every other MVP of this year (full roster to be announced soon) and the whole DataStax for such a great and well organised event.

## Tech Notes. Day 1

And after all this personal notes let's go into the technical notes I've made from the talks I've attended. Hope you find them interesting!!

> <http://esri.github.io/> project and it's [GIS tools for Hadoop subproject](https://github.com/Esri/gis-tools-for-hadoop)

This first one is a project to check out and bear in mind always when working with geospatial data and corresponding visualisations.

> Time series writing performance can be improved by buffering

This is pretty simple. If you do some buffering in your application and write bigger chunks of data, rather than doing one write per observation. It's likely that your overall performance goes up.

### PagerDuty's 'One year of Cassandra failures'

PagerDuty's talk about their infrastructure was really interesting to me, as it somehow resembles ours.

> Watch to dropped messages counts. They anticipate bigger issues
>
> Upgrade Cassandra. At least to the corresponding latest DSE's version.
>
> The Cassandra Danger Metrics page.

Having a Dashboard of danger metrics to look at can be very useful. This should include pending tasks and latencies at least.

### The Weather Company

[Robbie Strickland's](https://twitter.com/rs_atl) was one of my favourite ones. I found it really dense and intense. Lots of notes here.

![The Weather Company Architecture](/assets/media/weather_co_infrastructure.png)

First of all is to check Robbie's book [Cassandra High Availability](http://www.amazon.co.uk/Cassandra-High-Availability-Robbie-Strickland/dp/1783989122)

> Be careful when using DTCS. It is dangerous.

Compactions in Cassandra have to be deeply understood, otherwise they will simply bite you sooner or later.

> Process and filter your events at ingestion time.

That makes sense. Instead of having unstructured data, parse and process your data before ingesting it and that way you'll be able to filter invalid data if necessary. If you don't do it, you'll have to parse and process every time you read, which is definitely worse.

And speaking particularly on his lambda architecture (in the picture), a couple of ideas:

*   Daily backup data from C* to S3 + Parquet. C* writes quickly but is more expensive than S3 for long term storage. Compute analytics at C* and dump when data is not likely to be required anymore.
*   Beware versions: Spark + Cassandra = 2.1.8
*   The usage of secondary indexes to help Spark reading data is a good practice, but keeping index cardinality low (<= 50k)
*   Beware wide rows, it only takes one to get you in trouble. Use `nodetool toppartitions` or `nodetool cfstats Max row bytes`
*   Make your data visualisable ASAP: [Zeppelin project](http://zeppelin-project.org/)

## Tech Notes. Day 2

### Netflix

[Christos Kalantzis](https://twitter.com/chriskalan) and his colleagues' talk about how they survived AWS re:boot gave as well lots of notes:

*   Run periodic checks on each nodes' health, using, for example, Jenkins.
*   They use internal products [Atlas](https://github.com/Netflix/atlas) and [Priam](https://github.com/Netflix/Priam) for managing and monitoring the cluster.
*   Have idempotent processes.
*   Retry with exponential back-off.
*   Collect data/stats on your clusters to predict failures.
*   No ops team. Everyone acts as devops and gets on-call.

### Troubleshooting with Cassandra

This talk from a DataStax's support engineer left several notes as well:

*   If cache is full but the hit rate is low, increase cache capacity.
*   If `memtableflushwriter:All time blocked` jobs is significant means disk pressure on writes.
*   `proxyhistograms` command shows the full time to process the request, including the network round-trip from the coordinator to the required replicas. By contrast, the `cfhistograms` command shows the local latency for a single replica.
*   Use `describecluster` for schema disagreements.
*   Log warns with 'Compacting large partition' message when if encounters a big partition that you should take care of.

### Lessons learnt with Cassandra and Spark

This talk by [Christopher Bradford](https://twitter.com/bradfordcp) from [OpenSourceConnections](https://twitter.com/o19s) gave me some architectural notes.

*   Their architecture is composed by Cassandra to store data, Spark to ETL and Solr to search. An example of this setup is in their github account: <https://github.com/o19s/Spark-Cassandra-Demo>
*   Build balanced models: spread data/load evenly across all nodes.
*   Use vnodes in small clusters. Single token nodes if you're big (Apple, Netflix, ...)
*   Have a look at [Metrics](https://dropwizard.github.io/metrics/3.1.0/): [Dropwizard](http://www.dropwizard.io/)'s Java profiler project.

### Extreme Cassandra Optimization.

I must admit this talk by [Al Tobey](https://twitter.com/AlTobey) was a bit out of reach for me. There were so many advanced optimisation tips that I can only link to his guide to come back to it when I get the appropriate level: [Al's Cassandra 2.1 tunning guide](https://tobert.github.io/pages/als-cassandra-21-tuning-guide.html)

### Repeatable, Scalable, Reliable, Observable: Cassandra

This talk by [Aaron Morton](https://twitter.com/aaronmorton) had sooooo many notes on how to monitor Cassandra that at a given point I decided to give up and simply grab the slides later to review them. REALLY GOOD!!! <http://www.slideshare.net/aaronmorton/cassandra-sf-2015-repeatable-scalable-reliable-observable-cassandra>

### Case Study: Troubleshooting production issues as a developer.

And that was my talk!! A live coding demo!! I have to massively thank to everyone who attended to it as that was the last slot and I acknowledge everyone was already kind of burnout of Cassandra and I also hope they enjoyed it and found it valuable.

Here you can see a video of the same talk during the [Cassandra London Meetup][5] and the slides as well:

*   [Slides](http://bit.ly/1P3ZFq4)
*   [Video](http://bit.ly/1LiUS4p)

And that's about it!! Hope this notes are as useful for you guys as they're for me and a HUGE THANKS to everyone who spoke at the Summit for sharing such a valuable knowledge.
