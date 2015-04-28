---
title: Claim your 10 seconds back!
author: carlos
date: "2015-04-27"
description: The story of how we fought a performance issue in our Cassandra cluster
tags: ["Cassandra", "Performance"]
categories: ["development", "profiling"]
---

A couple of weeks ago we released a feature and it's performance was unexpectedly poor and here I want to share the steps and tools used to get to the root cause of the problem.

To give a little bit of background I'll tell you that the feature was something really common nowadays: **Saving a bunch of time series in Cassandra**

### Step 1: Look for evidences in metrics ###

The natural first step I think everyone does is to look at the graphs (here we use graphite) and try to find evidences of what's going on.

In our case we had a very clear evidence that something was wrong as the time consumed in the process had increased by around 600% after the deploy!!

![Massive time processing increase](/assets/media/events_latency_increase.png)

But that means that not only something is wrong in the feature but also, and even more scary, in our whole process!!
How can such a poorly performing piece of code have reached production without anyone noticing it before?
Ok, we don't have performance tests as part of our CI process, but we test every feature in our pre-production environments before going
to production and that should have appeared there!!
That would have simply been unacceptable and processes are easy and strong here [@_MyDrive](https://twitter.com/_mydrive), so,
after scrolling a little bit along the graphs we found an explanation. The tests ran in the pre-production environment were ok!

![Graph showing timing taken on pre-production](/assets/media/events_latency_increase_with_labs.png)

Ok, so we have somewhere to look at: something on our production environment is performing poorly and, at first glance, our stats are not showing it at all!.

### Step 2: Profile suspicious code ###

At this step we use the fantastic [RubyProf](https://github.com/ruby-prof/ruby-prof) to wrap the suspicious code in a `RubyProf.profile` block and save the
results to analyse later.

```
require 'rubyprof'

results = RubyProf.profile do
  [Code to profile]
end

printer = RubyProf::GraphPrinter.new results
printer.print(File.new('/tmp/profiled-events-insert.txt', 'w'), min_percent: 2)
```

Reading the saved files I could clearly see that the time was going into the Cassandra related stuff
and made the assumption that the problem would be somewhere in the model/queries stuff.

I could have read a little more of the profiling result and will probably have saved some steps here, but
as we were issuing several thousands of inserts asynchronously and the first lines of the profiling report
were pointing to Cassandra everything looked crystal clear.

### Step 3: Trace queries ###

There's only one thing we're doing here: **INSERT** so...

```
cqlsh:carlos_test> TRACING ON
cqlsh:carlos_test> INSERT INTO ...

 activity                         | timestamp    | source       | source_elapsed
----------------------------------+--------------+--------------+----------------
               execute_cql3_query | 10:26:35,809 | 10.36.136.42 |              0
          Parsing INSERT INTO ... | 10:26:35,836 | 10.36.136.42 |          26221
              Preparing statement | 10:26:35,847 | 10.36.136.42 |          37556
Determining replicas for mutation | 10:26:35,847 | 10.36.136.42 |          37867
   Acquiring switchLock read lock | 10:26:35,848 | 10.36.136.42 |          38492
           Appending to commitlog | 10:26:35,848 | 10.36.136.42 |          38558
        Adding to events memtable | 10:26:35,848 | 10.36.136.42 |          38600
                 Request complete | 10:26:35,847 | 10.36.136.42 |          38926
```

Looking at this results something looks broken on parsing the query, and running the
same thing on our pre-production environment it clearly shows as something broken!

```
cqlsh:benchmark> TRACING ON
cqlsh:benchmark> INSERT INTO ...

 activity                         | timestamp    | source        | source_elapsed
----------------------------------+--------------+---------------+----------------
               execute_cql3_query | 10:27:40,390 | 10.36.168.248 |              0
          Parsing INSERT INTO ... | 10:27:40,390 | 10.36.168.248 |             75
              Preparing statement | 10:27:40,390 | 10.36.168.248 |            233
Determining replicas for mutation | 10:27:40,390 | 10.36.168.248 |            615
   Acquiring switchLock read lock | 10:27:40,390 | 10.36.168.248 |            793
           Appending to commitlog | 10:27:40,390 | 10.36.168.248 |            827
        Adding to events memtable | 10:27:40,390 | 10.36.168.248 |            879
                 Request complete | 10:27:40,391 | 10.36.168.248 |           1099
```

But, what could be so wrong with parsing a query?

### Step 4: Simplify the problem ###

At this point I decided to write [a small Ruby program](https://gist.github.com/calonso/a26da8aa23f88e59abf4) that:

1. Connects to the Cassandra cluster
2. Creates a test keyspace
3. Creates a column family within the test keyspace
4. Runs an insert like the one profiled above
5. Drop the test keyspace

and profile it all using `RubyProf` to try to spot something obvious.

Running this script in production showed something useful, more than 98% of the time was spent in `Cluster#connect` method!
Furthermore, almost 100% of the time inside that method was going to `IO.select`! Which means that the time is being wasted
outside Ruby itself.

Actually I could have saved some time, because this exact same thing was also clear in the first profiling I did in step 1,
but my premature assumption made me walk some extra unnecessary steps.

Ok, we have a new symptom, but no idea where to look at, so after some desperate and useless attempts like
tcpdumping the communications between the client and the cluster I decided to go back to the script I wrote and...

### Step 5: Enable Logging ###

Maybe this should have been the first, right?

But this was really revealing!, for some reason, the driver was trying to connect to four nodes when in our ring
we only have three!! Of course, the connection to this extra node was failing on timeout (10 seconds) and that
was the source of our poor performance!

A quick google with the symptoms as the query and ... ta-dah!! We're facing [a Cassandra bug](https://issues.apache.org/jira/browse/CASSANDRA-6053)!!

Quickly applied the fix and we were saving our 10 seconds per execution again.

### Conclusions ###

1. Measure everything
    * Metrics and monitoring let us know we were facing an unexpected performance issue
2. Keep calm, read to the end
    * On first profiling report I could have spotted that the issue was on `Cluster#connect` but due to my eagerness to find a fix I made a wrong assumption that meant more time.
3. Thank the community
    * Thanks to the community we have great tools such as:
        1. [Cassandra](http://cassandra.apache.org/). Thanks [Apache](https://www.apache.org/) and [DataStax](http://www.datastax.com/)
        2. [Graphite](http://graphite.wikidot.com/). Thanks everyone who made it possible.
        3. [RubyProf](https://github.com/ruby-prof/ruby-prof). Thanks all members of [RubyProf Organisation](https://github.com/ruby-prof)
    * Special thanks to [Patrick McFadin](https://twitter.com/patrickmcfadin) who we involved in this process through [a StackOverflow question](http://stackoverflow.com/questions/29302655) and he has always been helpful and offered further advice in a chat we held during Cassandra Day 2015 London.
4. Take advantage of learning opportunities!
    * These kind of *unexpected* situations normally push you out of your comfort zone and really tests your limits which is really good for your personal development.
