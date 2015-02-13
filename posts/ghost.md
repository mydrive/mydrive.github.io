---
title: GHOST CVE-2015-0235
author: gavin
date: '2015-02-02'
description: Patching the GHOST Vulnerability
tags: ["security", "ghost"]
categories : ["security"]
---

Last week saw the first brand-name bug of 2015 in the shape of GHOST
[CVE](https://bugzilla.redhat.com/show_bug.cgi?id=CVE-2015-0235),
[Explanation](http://ma.ttias.be/ghost-critical-glibc-update-cve-2015-0235-gethostbyname-calls/).

Using our automated config management tools (Chef) we patched all systems
operated by MyDrive within 24 hours of a patch being made available by our
operating system vendor, and then performed a rolling reboot of all systems as
a precautionary measure to ensure all services were using the patched library.
This was all completed by noon UTC on Weds 28th Jan 2015.

MyDrive Solutions' operational staff closely monitor security disclosure
announcements, and patched all systems immediately when patches were issued by
our operating system vendor. Our automated configuration management tools
allow us to apply security patches such as this across our entire
infrastructure in moments.

We continue to proactively monitor our suppliers and vendors for issues
arising from security announcements.

We do not believe any action is required from our customers.
