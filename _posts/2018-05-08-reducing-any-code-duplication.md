---
layout: default
title: "Reducing any code duplication"
date:   2018-05-08 00:00:00 +0000
tags: Clean-Code C-Sharp
image:
  src: "/assets/images/converging-parts-rainbow.jpg"
  title: "parts to combine"
---

# {{ page.title }}

## General observations
I'm a big fan of the ["don't repeat yourself" (DRY)](http://wiki.c2.com/?DontRepeatYourself) principle (as I've mentioned for instance [here](/2018/05/03/multiple-inheritance-of-DTOs-would-be-a-good-idea.html) a bit). IMO, this very principle should be respected from the very development start. At the same time, the principle is one of the most violated -- especially on the front-end. However, lower layer applications and libraries tend to adhere to the principle much more -- as it's much easier to "scare" product owners or managers with a prospect of maintaining and aligning two _identical_ (or even worse -- _kinda identical_) parts of the long-lasting code further on.
