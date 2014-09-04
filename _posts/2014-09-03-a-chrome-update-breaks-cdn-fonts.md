---
layout: post
title: “A Chrome Update Breaks CDN Fonts”
---

Last Friday afternoon, I was happily looking forward to a long Labor Day weekend. My [business partner][dodd] has left for the beach and I was handling customer service. I log into our production [MoonClerk][mc] app, and low and behold, all of our custom fonts were broken. Then the tweets came.

![Broken Fonts](/images/assets/broken-fonts.png)

I’m at a total loss. No deploy, nothing changed. It’s just broken. This started a 10 hour endeavor to figure out a fix for our specific situation. In case you happen to be in the same situation, hopefully this will help.

I realize after stumbling on [this post and seeing the Aug 28 update][chrome] that Google automatically updated Chrome and the new version stop allowing cross origin fonts from loading. Here is our situation:

* Heroku 
* Amazon CloudFront CDN
* SSL Only

I could find a number of helpful resources which got me close but none fully worked for my situation. This [solution][solution] was the closest and got me 90% of the way there.

After adding the [font_assets gem][fa], hitting the font directly on our site had the appropriate headers but the same file on CloudFront (even after cache invalidation) did not contain the headers.

I noticed that the CloudFront version was returning a 301 redirect which clued me into the last piece of the puzzle. After another couple hours of nosing around the AWS documentation and CloudFront site, I finally found what I though may be the issue.

There is a setting in CloudFront:

```
CloudFront Distributions > Origins > Origin Policy Protocol
```

It has two options: HTTP & Match Origin. I left the default (HTTP) when initially setting up my distribution. As it turns out, this tells CloudFront how to access the Origin server when getting a new version of a file. Although the request to our font file was using HTTPS, CloudFront was then using HTTP behind the scenes to retrieve the file and was getting redirected to the HTTPS version and losing the headers in the process. After changing this setting and invalidating the cache for the specific font files (for the 27th time), our fonts worked again.

[dodd]: http://doddcaldwell.com
[mc]: http://www.moonclerk.com
[chrome]: http://www.holovaty.com/writing/cors-ie-cloudfront/
[solution]: http://kennethjiang.blogspot.com/2014/07/set-up-cors-in-cloudfront-for-custom.html
[fa]: https://github.com/ericallam/font_assets