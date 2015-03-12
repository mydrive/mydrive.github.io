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
cover leave and swarm more effectively on tasks.

With this in mind we took most of the Software Engineering team to AWS's free
AWSome day, which is a one day event they hold a few times a year in London, and are
taking on tour around Europe, so we could learn what services we could be taking
advantage of already, and what was available to look to use in future.

The day started fairly slowly for us, being a team that are already actively using
AWS in production most of the introductory morning sessions were covering things
we were already familiar with, and felt a bit sales heavy, which is understandable,
There was an audience of 300 people, many of whom had never used it before.

The initial sessions did confirm the cost benefits in terms of billing costs and
maintenance time we could achieve by moving to more intensive use of managed services.

The afternoon sessions were much more useful and there are a few services I think
we can use, and a few potential projects for the next Hack Day.

Archiving old data to [Glacier](http://aws.amazon.com/glacier/) to save storage costs, we can consider DynamoDB as
an alternative to Cassandra or RDS for some simple key/value data structures,
we're already looking at replacing Redis with [ElastiCache](http://aws.amazon.com/elasticache/), but I think we can
also take advantage of it to cache some common requests. We might/could be able to
replace Hosted Chef with [OpsWorks](http://aws.amazon.com/opsworks/) in some areas.

On a personal note, I'm going to register for a personal AWS account and move a
bunch of old [static websites to AWS](http://docs.aws.amazon.com/gettingstarted/latest/swh/website-hosting-intro.html)
), for a greatly reduced cost (free for a year).
I've also registered for the [AWS Summit](http://aws.amazon.com/summits/london/)
