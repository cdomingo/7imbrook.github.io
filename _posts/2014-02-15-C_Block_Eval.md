---
layout: post
title: "C Block Evaluation"
tagline: "The shorthand and techniques I use to be a great cocoa developer."
description: "I'm gonna make my first post about style and readablity, and a gcc c extension called block evaluation. Now supported by clang due to the teams forward effort to support all of the features found in gcc. This is one of my favorite discoveries however minor it may seem. This is what I've seen when someone is updating a frame on a view."
category: articles
comments: true
tags: [objective-c, shortcuts, style]
image:
  feature: texture-feature-04.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
---

I'm gonna make my first post about style and readablity, and a gcc c extension called block evaluation. Now supported by clang due to the teams forward effort to support all of the features found in gcc. This is one of my favorite discoveries however minor it may seem. This is what I've seen when someone is updating a frame on a view.

```objective-c
...
CGFrame newFrame = self.button.frame;
newFrame.origin.y = 50;
newFrame.origin.x = 50;
[self.button setFrame:newFrame];
...
```

To anyone this is a perfectly sane way of doing this. It’s not the only one, you could use `CGRectOffset` if you are moving a view but if you know exactly where to put it you might as well set it. But this reads backward. The new frame is set up and then at the end you set it. At the end of these lines you figure out what they did and when there’s code around it is hard to figure out what code goes to what. What we’re allowed to do with the GCC C Block Evaluation is return code surrounded by `({ … })`. This will execute whatever is inside here and return the last line. Now take a look at this.

```objective-c
...
[self.button setFrame:({
    CGFrame frame = self.button.frame;
    frame.origin.y = 50;
    frame.origin.x = 50;
    frame;
})];
...
```
This now becomes much more readable. All the configuration for an object can be encapsulated inside this code block and before you even get to the block you know what it’s going to do. It also then exists in it’s own scope, which is nice because you don’t need unique variable names like `newButtonFrame` and `newButton2Frame`, overall this can really clean up your code.






