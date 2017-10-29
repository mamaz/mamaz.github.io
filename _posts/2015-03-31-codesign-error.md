---
layout: post
title: Strange issue with XCode 6.2
comments: true
---

Today I've found strange issue with XCode 6.2 as it won't run on device, event though the prov. profile and Certificates matches.

The error is:

"no identity found Command /usr/bin/codesign failed with exit code 1"

the codesign command uses different SHA-1 hash than the Certificate that I use.

I've tried everything including removing and inserting the Certificates, and refreshing prov. profile in Accounts > Apple ID > View Details.

It's working if I use Wildcard profile. But even though Certificate and the prov. profile match each other in Build Settings. The error is still occurred.

Then I decided to try to *re-download* the development prov. profile from Apple Portal even though I still have the same prov profile in my Mac.

Then it just works, I have no idea why.
