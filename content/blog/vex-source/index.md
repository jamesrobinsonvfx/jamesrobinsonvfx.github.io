---
title: "Vex Source"
date: 2021-09-10
draft: false
ShowReadingTime: true
ShowToc: true
summary: "Use one VOP network or VEX wrangle to drive many others without channel referencing"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Vex Source"
    relative: true
categories: ["houdini"]
tags: ["tips", "vex", "vops"]
---

> Hipfile: [jamesr_vexsource.hiplc](houdini/hip/jamesr_vexsource.hiplc)

Houdini offers several ways to duplicate and reuse nodes.

- Copy and paste nodes (**Ctrl + c**, **Ctrl + v**).
- Create an HDA.
- **RMB** > **Actions** > **Create Reference Copy**

...to name a few.

Sometimes, you might find yourself re-using the same VOP network or VEX wrangle without changing the internal nodes or code. But what happens if you want to change them all at once?

## Scenario

{{< video src="images/noisy-flippy.mp4" >}}

Let's say we have a VOP network that applies some noisy displacement animation to a character (see hipfile above). There are several other characters in the scene who need the same displacement effect applied to them. We have a few options:

### Option 1: Copy and Paste the VOP Network

{{< figure src="images/copy-paste.gif" title="" caption="Copy and Paste" alt="Copy and Paste" >}}

Simple as that. We can just duplicate it around and we're all set. 90% of the time this will probably be what you do day-to-day. The downside though is that anytime you update the internal nodes/code, you'll have to do the same thing to all the copies...

### Option 2: Create an HDA

HDAs are perfect for when you want to bundle up some nodes and reuse them all over the place. There are numerous advantages to using HDAs that I won't go into in this post, but sometimes creating and tracking an HDA is just a bit overkill. In our case, we have one VOP that makes some noise...we don't really need to go through the effort of creating an asset for it!

### Option 3: Create a Reference Copy

{{< figure src="images/reference-copy.gif" title="" caption="RMB > Actions > Create Reference Copy" alt="RMB > Actions > Create Reference Copy" >}}

We could create a **Reference Copy** which is really just like copying and pasting the node like in the first option, except the new pasted node has relative channel references back to all the parameters on the source node. This works just fine, and we get a copy of all the internal nodes as well. But even with the reference copy, if we were to change the internal nodes of the source VOP network like adding new nodes or removing old ones, those changes would
*not* be reflected in the copies.

### Option 4: Change the Vex Source

The last option we'll go over is really the whole point of this post. On each VOP network, there is a parameter called **Vex Source**. By default, this is is set to **Myself**, which means it uses the operators inside itself to do all the work.

{{< figure src="images/vex-source.png" title="" caption="Vex Source Parameter" alt="Vex Source Parameter" >}}

If we change it instead to **Shop**, the **Shop Path** parameter is then exposed, and we can actually set that parameter to *another VOP network in the scene*.

We leave this VOP network empty - its internals are being overridden by the contents of the VOP network specified in that **Shop Path** parameter!

#### Steps

{{< figure src="images/vopnet-shop-path-setup.gif" title="" caption="VOP Network Shop Path Setup GIF" alt="VOP Network Shop Path Setup GIF" >}}

To recap, the steps are as follows:
1. Create a VOP network that does something you want to reuse on other geo.
2. Create an empty VOP network and hook it up to wherever you want to reuse the original.
3. Select the original VOP network and hit **Ctrl + c**. This will copy the path
   to the node to the clipboard.
4. Select the empty VOP network, and set the **Vex Source** parameter to **Shop**.
5. Paste the copied path to the source VOP network in the **Shop Path**
   parameter (if you want this to be a relative path, that's fine too).

{{< figure src="images/change-source-vopnet.gif" title="" caption="Changes to the source network are propagated immediately!" alt="Changes to the source network are propagated immediately!" >}}

## What about Wrangles?

Hidden inside each wrangle is actually just a VOP network with a Snippet VOP inside!

VEX wrangles don't have the **Vex Source** parameter exposed at the top level, so we actually won't be able to use an empty wrangle on each of our other streams.

Instead, let's just write one wrangle with the code on it that we want to reuse, and the rest of the geometry we want to copy it around to will have empty *VOP Networks* instead, just like before.

{{< figure src="images/reference-wrangle.gif" title="" caption="Wrangle Source" alt="Wrangle Source" >}}

#### Steps:

1. Create the source wrangle with the desired code.
2. Create an empty VOP network and hook it up to the other geometry we want to process.
3. Dive inside the source wrangle node, and **Ctrl + c** on the VOP network inside. This copies the path to the node to the clipboard.
4. On each of the new empty VOP networks, set the **Vex Source** parameter to **Shop**.
5. Paste the path of the copied VOP network into the **Shop Path** parameter.

## Final Notes

The biggest downside to this method is that you can't really adjust the parameters on the copies. However, you could get around this by instead using attributes on the geometry to control certain parts of your setup!

{{< figure src="images/attribs-for-parms.gif" title="" caption="Attributes for Parameters" alt="Attributes for Parameters" >}}

### Channel Referencing Wrangles

Another pretty useful way of referencing another wrangle is to simply channel reference the **Snippet** parameter from another wrangle.

{{< figure src="images/channel-ref-snippet-string.png" title="" caption="Channel Reference Snippet String" alt="Channel Reference Snippet String" >}}

This way is also useful because you can add parameters to the wrangle copy's interface and update those, and they *will* get picked up and used while your code lives on just a single wrangle.

(Thanks Daniel)

### Evaluation Node Path
However, if you still want the source wrangle's parameter sliders to affect the copies, you'll need to update the **Evaluation Node Path** parameter under the **Bindings** tab of the wrangle copies.

{{< figure src="images/eval-node-path.png" title="" caption="Evaluation Node Path" alt="Evaluation Node Path" >}}

This tells the wrangle to look at *that* node instead when trying to figure out where `ch()` parameters are.

{{< figure src="images/eval-node-path-in-action.gif" title="" caption="Evaluation Node Path in Action" alt="Evaluation Node Path in Action" >}}
