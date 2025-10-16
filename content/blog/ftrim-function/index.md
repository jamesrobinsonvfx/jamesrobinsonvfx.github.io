---
title: "ftrim() HScript function"
date: 2021-08-19
draft: false
ShowReadingTime: true
summary: "Trim unwanted digits from parameter values"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Trim excess digits"
    relative: true
categories: ["houdini"]
tags: ["houdini", "tips", "hscript"]
hipfile: houdini/hip/jamesr_ftrim.hiplc
---

{{< attachments >}}

## Problem
Sometimes you want to reference the value of a parameter and display it as a string to put in a Font SOP, or the Viewport Comment of a Camera node when you're wedging sims or making some sort of visualizer.

But quite often, if you're referencing a `float` parameter, you wind up getting allllll the digits that come with it, full precision and all, rather than just the nice value you see in the interface.

{{< figure src="images/problem.png" title="" caption="Super long string of numbers when channel referencing" alt="Super long string of numbers when channel referencing" >}}

ie.
```
0.04
```
becomes
```
0.040000000000000001
```
which is probably *not* what you want!

A possible solution to trim off some of the extra digits might look something like

```
floor(ch("/some/parm") * 1000)/1000
```
Unfortunately, this fails too :(

## Solution

The solution is actually quite simple! We can use the `ftrim()` function from HScript. `ftrim()` will strip off all those unwanted digits and leave you with a nice clean value, pretty much as you typed it!

```
Some Parameter Value: `ftrim(ch("/some/parameter"))`
```

{{< figure src="images/solution.png" title="" caption="Using `ftrim()` to shore it up" alt="Using `ftrim()` to shore it up" >}}

Of course, this also works in a **Font SOP** too.

{{< figure src="images/font-sop.png" title="" caption="Demo on a Font SOP" alt="Demo on a Font SOP" >}}

## *Update 9 Jan 2022*
Here's a handy snippet for programmatically wrapping channel references in
`ftrim()`. Taken from the [Linewriter](https://github.com/jamesrobinsonvfx/linewriter) tool.

<script src="https://gist.github.com/jamesrobinsonvfx/f17a0ec451428fbe71f9e58c1800225f.js"></script>
