---
title: Ruby Thread Synchronisation
author: Carlos Alonso
date: "2015-01-29"
description: Little advice on how to synchronize threads in Ruby when using the Publish-subscribe messaging pattern by using end of operation objects.
tags: ["Ruby", "Threading", "Publish-subscribe", "Messaging", "Patterns", "Synchronisation"]
categories: ["development", "programming"]
---

Multi-Threading programming is one of those things that I like doing from time
to time. I find it particularly useful when automating any task that involves
network downloading.

My approach is usually a [Publish-subscribe](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) using the very handy Ruby's
[`Queue` class](http://ruby-doc.org/stdlib-2.0/libdoc/thread/rdoc/Queue.html).

Sometimes I build it using several stages, i.e. last time I used it I was building
a tool for downloading and processing a whole S3 bucket and I designed it with
two messaging layers. First one publishes object names within the bucket
for the second one to pick them and actually download and process them and finally
publish the results for an aggregator thread (the main program's thread).

![2 Stages Pub Sub Diagram](/media/2_stages_pub_sub_diagram.png)

My programming approach to this is leveraging all synchronisation in queues rather
than waiting for threads or passing messages between them.
But don't worry, that's not a crazy or hacky approach at all, is just the Ruby's
recommended way.

So, what I'm going to do here is just explain by an example how to properly do
it avoiding errors and obtaining a clean end. Let's start off by showing a
failing example.

```ruby
require 'thread'

queue = Queue.new

producer = Thread.new {
  5.times do |i|
    queue << i
    sleep 1
  end
  p 'Producer exiting'
}

consumer = Thread.new {
  while producer.alive?
    p "Popped #{queue.pop}"
  end
  p 'Consumer exiting'
}

producer.join
consumer.join
```

This code sets up two threads, a publisher(producer) and a subscriber(consumer).
The producer publishes a value to the queue and sleeps a second for 5 times.
The consumer simply pulls messages from the queue as soon as they're available.

The producer exit condition is very straightforward. As soon as it finishes it's
job, it simply finishes. The consumer, on his end, monitors the producer status
and will exit as soon as it detects the producer is not alive anymore.

Finally, the main thread waits for both to finish[`Thread.join`](http://www.ruby-doc.org/core-2.2.0/Thread.html#method-i-join).

All looks good, doesn't it? But when we run it... Crash!!
```
"Popped 0"
"Popped 1"
"Popped 2"
"Popped 3"
"Popped 4"
"Producer exiting"
`join': No live threads left. Deadlock? (fatal)
```

Investigating a bit you'll find that this error is raised at the `queue.pop` consumer's
invocation. That's because when it checked the status of the producer, it was still alive.
Now we could try several approaches but the best and most robust one I think it is to use
what I call 'end of operation objects'.

Those objects are simply instances of a dummy class which purpose is to signal the end
of the operations queue. Using end of opertion objects we could rewrite our piece
of code as follows:

```ruby
require 'thread'

class EndOfOp ; end

queue = Queue.new

producer = Thread.new {
  5.times do |i|
    queue << i
    sleep 1
  end
  p 'Producer exiting'
  queue << EndOfOp.new
}

consumer = Thread.new {
  while obj = queue.pop
    break if obj.instance_of? EndOfOp
    p "Popped #{obj}"
  end
  p 'Consumer exiting'
}

producer.join
consumer.join
```

Now the producer pushes an instance of the `EndOfOp` dummy class just before exiting
to signal the consumer that it has finished its job. The consumer, on his end just
tests every pulled object that it is not an `EndOfOp` in order to continue.

Executing this code we would see:
```
"Popped 0"
"Popped 1"
"Popped 2"
"Popped 3"
"Popped 4"
"Producer exiting"
"Consumer exiting"
[Finished in 5.1s]
```

And that's all. Happy pubsubbing!!
