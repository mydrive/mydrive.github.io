---
title: Security Updates (POODLE and OpenSSL)
author: gavin
date: '2014-10-17'
description: Security patches for POODLE and others applied in recent days
tags: [security, ops, POODLE, SSL]
categories : ["security"]
---

A couple of security vulnerabilities were announced this week which are worth noting.

The first was a vulnerability in SSL V3, which was nicknamed
[POODLE](http://googleonlinesecurity.blogspot.co.uk/2014/10/this-poodle-bites-exploiting-ssl-30.html)
by the researchers who discovered it. This protocol is 18 years old, but was
still widely supported. After the announcement, MyDrive removed support for SSL
v3 from all web servers and load balancers. Updates were completed on
Wednesday 15th October 2014.

The second notable vulnerability was in
[OpenSSL](http://www.ubuntu.com/usn/usn-2385-1/). All servers were patched and
affected servers or services rebooted on Thursday 16th October 2014. This patch
also provided additional protection against the POODLE bug.

MyDrive Solutions' operational staff closely monitor security disclosure
announcements, and patched all systems immediately when patches were issued by
our operating system vendor. Our automated configuration management tools
allow us to apply security patches such as this across our entire
infrastructure in moments.

We continue to proactively monitor our suppliers and vendors for issues
arising from Shellshock and otherwise.

We do not believe any action is required from our customers.
