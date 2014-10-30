---
title: Ruby on Rails CVE-2014-7818
author: gavin
date: '2014-10-30'
description: Security patches for Ruby on Rails
tags: [security, ops, ruby, rails]
categories : ["security"]
---

A vulnerability in a Ruby on Rails component was [announced earlier today](https://groups.google.com/forum/#!topic/rubyonrails-security/dCp7duBiQgo).

We have determined that MyDrive's applications were not vulnerable to attack as
we do not serve static assets through rails, instead we use our web server
(nginx). Nevertheless We have patched all production services using Ruby on
Rails.

No action is required from our customers.
