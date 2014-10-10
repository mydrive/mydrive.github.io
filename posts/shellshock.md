---
title: Shellshock - MyDrive's Actions
author: gavin
date: "2014-10-07"
description: How MyDrive Solutions responded to the Shellshock bug
tags: ["security", "ops", "shellshock"]
categories : ["security"]
---

Between 24th and 27th September 2014 a number of vulnerabilties were exposed in
the `GNU bash` software installed on most computers running Linux, including:

[CVE-2014-6271](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271)

[CVE-2014-7169](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7169)

[CVE-2014-7186](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7186)

[CVE-2014-7187](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7187)

These vulnerability could allow attackers to execute arbitrary code on remote
servers, was given the nickname
[Shellshock](http://en.wikipedia.org/wiki/Shellshock_\(software_bug\)),
and was widely reported in both technical and mainstream media.

Based on our analysis of the vulnerability in context of the services we
operate and our system configuration we do not believe our infrastructure was
susceptible to attack using this vulnerability.

MyDrive Solutions' operational staff closely monitor security disclosure
announcements, and patched all systems immediately when patches were issued by
our operating system vendor. Our automated configuration management tools
allow us to apply security patches such as this across our entire
infrastructure in moments.

Patches were applied to all MyDrive servers on 24th, 25th, and 27th September
as they were issued by our vendor.

We continue to proactively monitor our suppliers and vendors for issues
arising from Shellshock and otherwise.

We do not believe any action is required from our customers.
