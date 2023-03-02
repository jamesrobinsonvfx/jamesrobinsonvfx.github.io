---
title: "Hair Cards to Guide Curves"
date: 2021-09-08
draft: false
ShowReadingTime: true
ShowToc: false
summary: Convert hair card geometry (like from a game engine) to guide curves
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Hair Cards to Guide Curves"
    relative: true
categories: ["houdini", "tools"]
tags: ["houdini", "tools", "hair"]
repo: cards_to_curves
---

## Overview

A few weeks ago, my friend and co-worker [Omar Taher](https://www.artstation.com/omartaher) came to me with an issue: he had a hair groom made of triangulated cards, and wanted to extract centerline curves from them to use as guides.

## Get it
Download the latest release [here](https://github.com/jamesrobinsonvfx/cards_to_curves/releases/latest/download/cards_to_curves.zip).

{{< attachments >}}

Inside the `houdini18.5/hda` folder you can grab the HDA and put it someplace where Houdini will find it. Otherwise, you can install it as a [Houdini Package](https://www.sidefx.com/docs/houdini/ref/plugins.html):

1. Download the release zip
2. Navigate to your Houdini user preferences folder (ie. `~/houdini18.5`)
3. Create a folder called `packages` if it doesn't exist yet
4. Copy the downloaded zip into `packages`
5. Extract the contents, and delete the zip archive if you want
6. Drag the file `cards_to_curves.json` up one level into to that `packages` folder

There is also a demo file located at `houdini18.5/hip` with some usage examples.

See the the help card on the node for more detailed info.


## Process
Sounds easy enough at first, but given some wonky polygon winding and shapes, it took a little extra work to make sure it was stable on every card.


### First Attempt

At first, not knowing much about how groom artists create hair cards, I took a VEX approach that matched up pairs of points based on their neighbors, and added a point at their center.

Unfortunately, that only worked when the cards were made with one row of alternating triangles, and after downloading a few more free hair card grooms, I realized that this method fails spectacularly if the artist created it differently.

### Final

Luckily, one thing all good hair card grooms have in common: they have nice UVs!
Should have thought of that sooner! This makes finding the centerline a breeze:
1. Evaluate the connectivity.
2. Split geo by UV seams, and promote `uv` attribute to points (if it isn't
   there already).
3. **Swap** `P` for `UV`, so the points now look like a UV layout, but in world
   space near the origin in the viewport.
4. Iterate over each connected piece (card), and create a line down the center
   of the bounding box, from the top to the bottom.
5. Add some more points to these lines with a **Resample**.
6. **Ray** the lines to the flattened hair card geo.
7. Use an **Attribute Interpolate** to put them back into world space.

Checkout the hipfile to see the setup, and crack open the hda for a deeper look.

## Tool Features

As mentioned above, this node will create guide curves from hair cards.

{{< figure src="images/cycle-grooms.gif" title="" caption="Some different grooms" alt="Some different grooms" >}}

By default, the tool is pretty straight forward. Plug in the static cards into the first input, get curves out! As long as you have a `uv` attribute (that works) you should get something useable.

### Single Card & Guide Geometry

{{< figure src="images/single-card.gif" title="" caption="Single Card" alt="Single Card" >}}

Enable the **Single Card** parameter to scrub through the cards one at a time for a quick debugging visualization.

### Stick to Animation

If a second input is connected, the guides will interpolate attributes (including position) from that geometry stream.

> Make sure the topology matches the original cards from the first input!

{{< figure src="images/stick-to-anim.gif" title="" caption="Stick to Animation" alt="Stick to Animation" >}}

{{< figure src="images/exploded-view.gif" title="" caption="Exploded View" alt="Exploded View" >}}

### Caveats

If you're getting bad guides that zig zag, or look like they're spanning across multiple guides, make sure the input cards are clean. There is a **Poly Doctor** embedded in the node that should help clean up non-manifold geometry, but sometimes it can't do all the work! Make sure that any problem cards are actually being treated as separate connected islands per-card. Otherwise either fix them, or delete them!

## Conclusion
Enjoy! If you have any problems, please feel welcome to add an issue to the github repo, or reach out to me directly.

{{< figure src="images/change-hair.gif" title="" caption="Hairstyles" alt="Hairstyles" >}}
