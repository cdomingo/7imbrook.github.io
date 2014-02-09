---
layout: post
title: "What makes objective-c my language"
tagline: "The shorthand and techniques I use to be a great cocoa developer."
description: "As an avid objective-c developer I have become accustomed to the ways of the smalltalk style object system and the power of the runtime. For the past three years I’ve learned something new every week I’ve spent nose deep into projects. Everything from powerful language features to simple shortcuts, code style, and documentation. Techniques for working with teams and how to keep your teammates sanity in check. I want to share with you how I work."
category: articles
tags: [welcome, objective-c, shortcuts, style]
image:
  feature: texture-feature-04.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
---

As an avid objective-c developer I have become accustomed to the ways of the smalltalk style object system and the power of the runtime. For the past three years I’ve learned something new every week I’ve spent nose deep into projects. Everything from powerful language features to simple shortcuts, code style, and documentation. Techniques for working with teams and how to keep your teammates sanity in check. I want to share with you how I work.

I want to start with style, and a gcc c extension called block evaluation, now supported by clang due to the teams forward effort to support all of the features found in gcc. This is one of my favorite discoveries however minor it may seem. Take a look at the following lines of code setting up a button.

```objective-c
_saveButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    [_saveButton setTitle:@"Done" forState:UIControlStateNormal];
    [_saveButton addTarget:self action:_doneButton forControlEvents:UIControlEventTouchUpInside];
    [self.view addConstraints:contraintsForButton];
    [self.view addSubview:_saveButton];

```