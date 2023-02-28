---
title: "Multi-Output Display Hotkeys"
date: 2021-12-13
draft: false
ShowReadingTime: true
ShowToc: true
summary: "Quickly switch the display for nodes with multiple outputs (Vellum, Split, RBD, etc.) without a Null"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Output for View Flag"
    relative: true
categories: ["houdini"]
tags: ["tips", "hotkey", "ui", "output"]
---

## Overview

In the past few releases, SideFX has been going pretty hard with the multi-output workflow style nodes, particularly with Vellum and RBD SOPs.

The most common way to view the outputs of these nodes is to just drop a null down and display it.

{{< figure src="images/drop-a-null.png" title="" caption="Drop a Null" alt="Drop a Null" >}}

This can be useful for other things, but sometimes you just want a quicker way, with less clutter!

## Output for View

{{< figure src="images/rmb-menu.png" title="RMB > Flags > Output for View" caption="Output for View Flag" alt="Output for View Flag" >}}

Right-click menu to the rescue! You can actually set the output you want to view by changing a flag from the menu.

***This will not affect the actual output result of the node!*** This is important to note, because it only affects the display, not what is actually being passed through the outputs.

## Assigning a Hotkey

Since this option is in a menu (like a lot of other useful options!), we can actually bind this to a hotkey.

{{< figure src="images/hotkey-editor.png" title="" caption="Hotkey Editor" alt="Hotkey Editor" >}}

1. Open the hotkey editor
    > Edit > Hotkeys
2. Search for Set View Output
3. Assign a hotkey for outputs 1 - 4

> On Linux/Windows, I find that Ctrl + Alt + 1 (1 through 4) works nicely.

> This post was written on a MacBook, so here it’s just Option + Cmd + 1-4

Now, changing the displayed output is as easy as selecting the node and cycling through your new hotkeys!

{{< video src="images/cycle-output-display.mp4" >}}

## Bonus: Python

If you're interesting in doing something scripty with this, you can use the following snippet to change the output:

```python
# Outputs are a zero-based index
node = hou.node("/obj/geo1/vellumtetrahedral")
node.setOutputForViewFlag(1)
```
