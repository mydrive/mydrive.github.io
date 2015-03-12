---
title: AWS AWSome Day
author: mike
date: "2015-03-12"
description: Learning more about using AWS for all the things
tags: ["AWS", "Amazon", "Cloud", "RDS", ]
categories: ["development", "operations", "aws", "devops"]
---

![AWSome Day banner](/assets/media/awsome_day.jpg)


We make extensive use of AWS infrastructure [@MyDrive](https://twitter.com/_mydrive).
We've been using them for a long time, initially using them to mirror a more traditional
infrastructure using cloud resources, to allow portability between cloud providers.

More recently we've identified that we can make considerable cost and time savings
if we start using more of the baked in AWS Managed Services and we've begun that transition,
for example we have already moved our realational database infrastructure to [RDS](http://aws.amazon.com/rds/).

We've also been cross skilling Developers and Operations staff to allow us to
cover leave and swarm more effectively on tasks. My background is as a Developer 
so I was looking forward to learn more about what services we could use to allow us
to deliver faster, and more consistency. I'm also on our Pager Duty rota, so I'm also
obviously very interested in services that make the operational maintenance someone
elses problem!

With this in mind we took most of the Software Engineering team to AWS's free
AWSome day, which is a one day event they hold a few times a year in London, and are
taking on tour around Europe, so we could learn what services we could be taking
advantage of already, and what was available to look to use in future.

The day started fairly slowly for us, being a team that are already actively using
AWS in production most of the introductory morning sessions were covering things
we were already familiar with on the computational and networking layers. It also
felt a bit sales heavy, which is understandable, given it was a free event and
many of the attendees were new to AWS. 

The afternoon sessions were much more useful and got into the meet of what we were 
interested in, focusing on many of the managed services we don't use and there are
a few services I think we can use, and a few potential projects for the next Hack Day.

Archiving old data to [Glacier](http://aws.amazon.com/glacier/) should save us a bit
amount on storage costs. Although the savings are small, it's so easy to set up 
automated archiving rules on a per bucket basis that there's almost no cost to
implement archiving. 

I hadn't looked much at [DynamoDB](http://aws.amazon.com/dynamodb/) but it does seem like it might be a good addition to
our database infrastructure. It's a simple No-SQL key, value pair store. With each
object limited to 400KB. One nice feature is that you can configure your required
IOPS so it's trivial to scale. It's also comfortable with updates, and supports 
secondary indexes. There are certainly some small, high throughput, simple data sets
where DynamoDB sounds like it would be a better fit than Cassandra or a Relational
Database.

We're already looking at replacing Redis with [ElastiCache](http://aws.amazon.com/elasticache/), but I think we can
also take advantage of it to cache some common requests.

They didn't go into [OpsWorks](http://aws.amazon.com/opsworks/) in much detail on the
day but from my initial research it looks like We might/could be able to take advantage
of it to replace hosted Chef in some areas.

On a personal note, I'm going to register for a personal AWS account and move a
bunch of old [static websites to AWS](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html)
), for a greatly reduced cost (free for a year).

I've also registered for the [AWS Summit](http://aws.amazon.com/summits/london/) which
looks to have some good hands on technical sessions. 
