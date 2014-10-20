---
layout: post
title: "What I write about when I have nothing to write about"
date: 2014-07-17 08:38:31 -0400
comments: true
categories: 
---


{% img http://placekitten.com/890/280 %}


Randomness: A tool that looked interesting: REVEAL.

<img src="{{ root_url }}/images/reveal-SS.png" />

30 day free trial; this tool will make reviewing complex views much simpler!


Chaos: A way to use UIActivityIndicators with AFNetworking.

```objc
#import "UIActivityIndicatorView+AFNetworking.h"

[self.activityIndicator setAnimatingWithStateOfTask:task];
```

This automates the use of the activity indicator to start and stop as needed, instead of keeping a count of the number of operations that are downloading/uploading data!


Entropy: How to show a photo in Octopress that you have on your local machine:

1) First, put it into /source/images within the Octopress folder.

2) Use a PNG (maybe you can use other formats, but PNG should definitely work.)

3) Use classic HTML, ignore the format shown on the Octopress website!


```html
<img src="{{ root_url }}/images/[Name of file]" />
```




