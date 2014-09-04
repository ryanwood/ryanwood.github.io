---
layout: post
title: A Chrome Update Breaks CDN Fonts
---

Last Friday afternoon, I was happily looking forward to a long Labor Day weekend. My [business partner][dodd] has left for the beach and I was handling customer service. I log into our production [MoonClerk][mc] app, and low and behold, all of our custom fonts were broken. Then the tweets came.

![Broken Fonts](/images/assets/broken-fonts.png)

I’m at a total loss. No deploy, nothing changed. It’s just broken. This started a 10 hour endeavor to figure out a fix for our specific situation. In case you happen to be in the same situation, hopefully this will help.

I realize after stumbling upon [the Aug 28 update][chrome] and it's [source][chrome2] that Google automatically updated Chrome and the new version stop allowing cross origin fonts from loading. Here is our situation:

* Rails 4 app hosted on Heroku
* Assets on Amazon CloudFront CDN
* SSL Only Access

Most of the suggestions were to update Apache or nginx to serve up the appropriate headers (Access-Control-Allow-Origin) for the font files. Unfortunately, there is no web server to configure on [Heroku][heroku]. That leave us with a Rack middleware solution. Thanks to [Kenneth][solution] I learned about the [font_assets gem][fa]. Awesome solution.

Adding the [font_assets gem][fa] got me close but it still wasn't working. Requesting the font on my server had the appropriate headers but the same font request via CloudFront (even after cache invalidation) did not contain the headers.

Then I noticed that the CloudFront version was returning a 301 redirect which clued me into the last piece of the puzzle. After another couple hours of nosing around the AWS documentation and CloudFront site, I finally found what I though may be the issue.

There is a setting in the CloudFront AWS console:

```
CloudFront Distributions > Origins > Origin Policy Protocol
```

It has two options: HTTP & Match Origin. I left the default (HTTP) when initially setting up my distribution. As it turns out, this tells CloudFront how to access the Origin server when getting a new version of a file. Although the request to our font file was using HTTPS, CloudFront was then using HTTP behind the scenes to retrieve the file and was getting redirected to the HTTPS version and losing the headers in the process. After changing this setting and invalidating the cache for the specific font files (for the 27th time), our fonts worked again.

Hopefully it won't take you 10 hours.

[dodd]: http://doddcaldwell.com
[mc]: http://www.moonclerk.com
[chrome]: http://www.holovaty.com/writing/cors-ie-cloudfront/
[chrome2]: https://coderwall.com/p/ub8zug
[solution]: http://kennethjiang.blogspot.com/2014/07/set-up-cors-in-cloudfront-for-custom.html
[fa]: https://github.com/ericallam/font_assets
[heroku]: http://heroku.com