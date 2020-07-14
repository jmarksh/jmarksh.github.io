---
layout: post
title: Do Not Repeat Yourself
category: Coding
tags: Jekyll HTML CSS
author: Josh
---

I'm a slow coder. That's fine at the moment -- it's not my day job and no one is paying me to do it. I enjoy going step-by-step and seeing the results and tweaking this and that. What I **despise** doing, though, is over-coding.

One of my more-recent forays into coding found media queries which seemed to be **the** solution for responsive web design. At least that was the use-case, anyway. It never sat right with me. Create re-written sections of code for every possible window size and layout direction? Oh, screw that.

Luckily, it seems, with a little thought (and a few new developments in CSS), people have gotten on board with other ideas, like using `min()`, `max()`, `clamp()` (or `max(x, min(y, z))` anyway), and mercifully `minmax()` in grid.

I, for one, code outside-in, and the layout is extremely important for me to get right. (Who needs content, right?) Grid has just been amazing.

Site update: Not much has changed in this short time with my birthday intervening *hiccup*, but there's been some formatting inspired by skeleton.css and some data importing with my plants database in a .csv -- never done that one before.

![A screenshot of new formatting on the home page.](/assets/img/site1-1.jpg)

![A screenshot showing colors in the plants database.](/assets/img/plants1-1.jpg)
