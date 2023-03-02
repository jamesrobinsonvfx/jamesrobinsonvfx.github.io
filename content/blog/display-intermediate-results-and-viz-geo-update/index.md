---
title: "Display Intermediate Results and Viz Geo [Update]"
date: 2023-02-15
draft: false
ShowReadingTime: true
ShowToc: true
summary: "A minor improvement on the previous post"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Slightly nicer way to output using a negative index"
    relative: true
hipfile: houdini/hip/jamesr_displayintermediateresultsandvisualizationgeoupdate.hiplc
---

{{< attachments >}}

{{< video src="images/demo-menu.mp4" >}}

# Overview

One thing I didn’t realize until after I posted the last [blog post/video](blog/display-intermediate-results-and-visualization-geo/) is that you can actually add an output with a **negative index**! This output won’t show up as a port on the node.

{{< figure src="images/negative-output-index.png" title="" caption="Negative Output Index" alt="Negative Output Index" >}}

We can see this on the **RBD Configure SOP**. When we hit Enter in the viewport to enter the node’s Viewer State, we are switched to viewing the `-1` output.

{{< figure src="images/rbd-configure-output.png" title="" caption="RBD Configure Negative Output" alt="RBD Configure Negative Output" >}}

{{< figure src="images/rbd-configure-contents.png" title="" caption="RBD Configure Contents" alt="RBD Configure Contents" >}}

We can’t actually hook up to this output in the network editor, which could make this an excellent option since it causes less clutter!

# Example

Let’s implement something similar ourselves. We'll create a setup that takes some input geo and adds some point normals to it. It will have a couple of false-color visualization modes: **Height** and **Normal**.

When we switch our **Display** menu, if we select anything except the "final result" of the node let's switch to showing the `-1` output in the viewport (which will only be outputting the visualization geo).

1. Open the hipfile and see the attached setup (or copy it from the screenshots):

    {{< figure src="images/example-overview.png" title="" caption="Example Overview" alt="Example Overview" >}}

    {{< figure src="images/example-contents.png" title="" caption="Example Contents" alt="Example Contents" >}}

2. Create an **Ordered Menu** Parameter called `display` with the following token/value pairs:

    {{< figure src="images/menu-tokens.png" title="" caption="Menu Tokens" alt="Menu Tokens" >}}

3. Don't forget to link it up to your visualizer switch!

    {{< figure src="images/switch-minus-one.png" title="" caption="Switch (minus one)" alt="Switch (minus one)" >}}

    > We subtract `1` since we're using index `0` of the menu to select our actual output geo, and if we didn't subtract 1 our switch would be off by one.

4. Add a callback to the menu parameter that will switch to the `-1` output
   whenever we select one of the visualizer options.

    ```python
    kwargs["node"].setOutputForViewFlag(-1 if kwargs["script_value"] != "output" else 0)
    ```

    {{< figure src="images/menu-callback.png" title="" caption="Menu Callback" alt="Menu Callback" >}}


{{< video src="images/demo-menu.mp4" >}}

And that's it!
