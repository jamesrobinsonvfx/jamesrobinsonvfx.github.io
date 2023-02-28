---
title: "VDB Reshape SDF Close"
date: 2021-07-29
draft: false
ShowReadingTime: true
summary: "Faster way to dilate and erode an SDF"
cover:
    image: images/preview.jpg
    alt: "Cover Photo"
    caption: "VDB Reshape SDF (Close)"
    relative: true
categories: ["houdini"]
tags: ["tips", "houdini", "vdb", "sdf", "volume"]
---

> Hipfile: [jamesr_vdbreshapesdfclose.hip](houdini/hip/jamesr_vdbreshapesdfclose.hiplc)

### Dilate and Erode (Old)
Most people are probably familiar with the following workflow for sealing up gaps and holes in an SDF using 2 **VDB Reshape SDF** nodes:

1. Set the first one to **Dilate**
2. Set the second one to **Erode**
3. Channel reference the **Offset** parameter from the dilating node to the
   **Offset** parameter of the eroding node.
4. Adjust the offset until you're happy

> If you want to keep a filled interior, don't forget to set the **Trim**
> parameter to **None** (Houdini 18.5+)!

{{< figure src="images/old-way.jpg" title="" caption="Old way" alt="Old way" >}}

### Close (New)
Not sure when this was added (or maybe it has been here the whole time!), but there is another method that does the exact same thing in one go: **Close**.

{{< figure src="images/new-way.jpg" title="" caption="New way" alt="New way" >}}

### Conclusion
A side-by-side comparison of the two looks like you get the same result!

{{< figure src="images/side-by-side.gif" title="" caption="Side by side comparison" alt="Side by side comparison" >}}
