---
title: "Display Intermediate Results And Visualization Geo in an HDA"
date: 2023-02-01
draft: false
ShowReadingTime: true
summary: "Different methods for switching between visualization geo in an HDA or subnet"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Switching visualization geo in an HDA"
    relative: true
categories: ["houdini", "python"]
tags: ["houdini", "ui", "python", "hda", "subnet", "tools"]
ShowToc: true
---

> Hipfile: [jamesr_displayintermediateresultsandvisualizationgeo.hip](houdini/hip/jamesr_displayintermediateresultsandvisualizationgeo.hiplc)

{{< youtube "tEfFO4VoslY" >}}

# Overview

If you've ever built an HDA in Houdini, you might have come across the following problem:

*How can I switch between showing the final result and some visualization geo inside my HDA?*

Displaying intermediate results or visualizing stuff inside an HDA can be a little tricky in Houdini, mainly since changing the Display Flag in a locked asset isn't allowed, and oftentimes the built-in wireframe [Guide Geometry](https://www.sidefx.com/docs/houdini/ref/windows/optype.html#node) option isn't enough for the job.

I've run into this online and in my own work, and wanted to share a couple of ways you can easily display intermediate steps or visualization geometry in the viewport without too much fuss!

We'll cover three solutions:

- [Simple Switch SOP controlled by an Ordered Menu](#simple-switch-sop-and-an-ordered-menu-parameter)
- [Additional Output](#additional-output)
- [Additional Output + Output for View Flag Callback](#favorite-output-for-view-flag)

Feel free to follow along using the hipfile above.

# Before We Get Started

## Recap Some Basics

- Whichever node has the [Display Flag](https://www.sidefx.com/docs/houdini/network/flags.html#sop-flags) set on it is the one that shows up in the viewport. The result of the node that it's on is passed through the first/leftmost output (when not using an [Output SOP](https://www.sidefx.com/docs/houdini/nodes/sop/output.html)).

    {{< collapse summary="More info" >}}
{{< video src="images/display-flag-demo.mp4" caption="With no **Output SOP**, the **Display** and **Render** flags determine the output of your node.">}}
    {{< /collapse >}}

- When using **Output SOPs** inside a subnet/HDA, the **Display Flag** is ignored, and the result that's piped into the **Output SOP** (whose index is zero) is what shows up in the viewport.

    {{< collapse summary="More info" >}}

{{< video src="images/display-flag-output-demo.mp4" >}}

{{< figure src="images/output-sop-index.png" title="" caption="Caption" alt="Caption" >}}

    {{< /collapse >}}

- HDAs/Subnets can have multiple outputs when using the **Output SOP**.

    {{< collapse summary="More info" >}}

{{< figure src="images/multiple-outputs.png" title="" caption="Multiple Outputs" alt="Multiple Outputs" >}}

    {{< /collapse >}}

- You can change the [**Output for View Flag**](/blog/multi-output-display-hotkeys) to switch which output is shown in the viewport.

    {{< collapse summary="More info" >}}
{{< figure src="../multi-output-display-hotkeys/images/rmb-menu.png" title="" caption="Output for View Menu" alt="Output for View Menu" >}}
    {{< /collapse >}}


- **Display Flag** can ***not*** be moved inside of a locked HDA, even with a Python callback.

    {{< collapse summary="More info" >}}

{{< figure height="300px" src="images/display-flag-permission-denied.png" title="" caption="Permission Denied (Interface)" alt="Permission Denied (Interface)" >}}

{{< figure src="images/display-flag-python-permission-error.png" title="" caption="Permission Denied (Python)" alt="Permission Denied (Python)" >}}

```python
node = hou.node("/obj/locked_asset/displayflagcallback")
node.node("torus").setDisplayFlag(True)

Traceback (most recent call last):
File "<console>", line 1, in <module>
File "/Applications/Houdini/Current/Frameworks/Houdini.framework/Versions/Current/Resources/houdini/python3.9libs/hou.py", line 79 410, in setDisplayFlag
    return _hou.SopNode_setDisplayFlag(self, on)
hou.PermissionError: Failed to modify node or parameter because of a permission error.  Possible causes include locked assets, takes, product permissions or user specified permissions
```

    {{< /collapse >}}

- You could get even fancier than this tutorial using [Python Viewer States](https://www.sidefx.com/docs/houdini/hom/python_states.html), but we won't be covering that here.

## Example Setup

To demonstrate, we'll be creating a super simple Voronoi fracturing HDA. The focus is definitely not on how this tool works, but how we can improve the user experience!

{{< figure height="750px" src="images/basic-example-setup-network-editor.png" title="" caption="Basic Example Setup" alt="Basic Example Setup" >}}

## "Intermediate Steps" / "Visualization Geometry"?

I've mentioned the terms **Intermediate Steps** and **Visualization Geometry** twice now, but I should explain a bit more clearly what I mean in terms of the example setup that we'll use for the rest of the tutorial.

### Intermediate Steps

When your HDA wraps up a workflow that does a few things in order (like our example), it can be helpful to be able to see the result of each step. In this example, we're building a basic fracturing HDA. For debugging purposes, it could be good to be able to inspect the noised-up volume and the scattered points. Creating these geometries is what I'm referring to as **Intermediate Steps**. They contribute to, but are not the final result.

### Visualization Geometry

It could also be nice to see an exploded view of all our pieces with different colors assigned to them (for this example...probably not *that* useful in practice, but let's use it here as a demo!). Since this doesn't really contribute to the final result at all and is just for debugging/inspection, I call it **Visualization Geometry**.

# Simple: Switch SOP and an Ordered Menu Parameter

First up, we have the simplest method: We'll use a **Switch SOP** to switch what our node outputs, and link it to an **Ordered Menu** Parameter.

---

#### Pros
- Simple to set up

#### Cons
- Easy to forget and accidentally leave the menu on the wrong selection and output the wrong thing

---

First, let's create some **Null SOPs** to use as "anchors" for each thing we want to switch between. This helps keeps things clean, organized, and flexible.

{{< figure height="750px" src="images/anchor-nulls.png" title="" caption="Anchor Nulls for each step" alt="Anchor Nulls for each step" >}}

Next we'll plug all our nulls into the switch in the same order that we want to have their corresponding items in the dropdown menu.

{{< figure height="750px" src="images/nulls-into-switch.png" title="" caption="Nulls plugged into switch" alt="Nulls plugged into switch" >}}

1. Final Result (Fractured Pieces)
2. Noised-up Volume
3. Points
4. Exploded / Colorized Pieces

> Another common technique is to create an **Object Merge** for each **Null** and put the switch off to the side to reduce clutter, but I'll keep just plugging the nulls straight into the switch for this example.
{{< figure src="images/object-merges-into-a-switch.png" title="" caption="Object Merges into a switch" alt="Object Merges into a switch" >}}

**Switch SOPs** are controlled by an integer parameter, so we can use an **Ordered Menu** to control the switch index, since they are essentially integer parameters themselves but with a more descriptive interface.

{{< figure src="images/ordered-menu-switch-sop.png" title="" caption="Ordered Menu linked to a Switch SOP" alt="Ordered Menu linked to a Switch SOP" >}}

{{< video src="images/ordered-menu-switch-fast.mp4" >}}
Creating a Menu Parameter for a Switch. Right Click > Open in New Tab for full rez version


> Check out [this post](/blog/evaluating-menu-parameters) for more info about how menus are evaluated.

# Additional Output

When you have one thing you'd like to visualize, a common technique is to add another **Output SOP** and connect it to your visualization geo. Whenever you want to view this visualization geo, you can just drop a **Null SOP** down, connect it to the extra output, and put the display flag on it (or use the [**Output for View Flag**](/tips/2021/12/13/multi-output-display-hotkeys)).

{{< video src="images/additional-output.mp4" >}}

---

#### Pros
- Less prone to error - we aren't changing the result of the primary output of the node
- We can flipbook the viz geo or view it side-by-side...anything like that

#### Cons
- Requires the user to drop another node just to see the viz geo, or switch the
  **Output for View Flag** themselves
- Data from this output might not really be useful elsewhere
- Could add clutter to your node if you already have many outputs

---

Since we have another output now, we can keep the final result flowing through the first output, so our users won't get tripped up by accidentally leaving the output to something that was only meant for visualization purposes.

For switching between multiple visualization items, we can have just one menu on our interface somewhere called "Visualization Output" or something, and that can control what comes through our extra output.

{{< figure src="images/additional-output-viz-menu.png" title="" caption="Visualization menu" alt="Visualization menu" >}}

# Favorite: Output for View Flag Callback

Let's take the [additional output](#additional-output) method above one step further and make it a bit slicker. It's kind of annoying to either make a **Null** each time you want to view the visualization output, or click through the **Output for View** RMB menu (especially if you haven't [setup hotkeys for that](/blog/multi-output-display-hotkeys)), so let's add a short callback to the menu parameter and do it automatically!

This is my favorite method because there's no risk of accidentally leaving a menu on the wrong thing and passing the wrong output (not that I've ever done this in production and kicked it to the farm...). Also, it doesn't require the user to muck around with creating another node just to see their visualization geo.

> More info on [Node Flags](https://www.sidefx.com/docs/houdini/network/menus.html#node-context-menu) from the docs.

---

#### Pros
- Always passing through the same result from the primary/first output
- No need to put down any extra nodes, or know anything about how to change the **Output for View Flag**
- All the pros of the previous methods

#### Cons
- Requires a little bit of Python

---

## Setup

First, we'll add another menu to control whether we're displaying the final output geo (the fractured pieces) or the extra visualization geo.

{{< figure src="images/second-menu.png" title="" caption="Second Menu" alt="Second Menu" >}}

And let's tighten up the two menus to make it a bit prettier:

{{< video src="images/tighten-it-up-2x.mp4" caption="Tighten it up">}}

> Optionally, we *could* add a [disable/hide when rule](https://www.sidefx.com/docs/houdini/ref/windows/optype.html#conditions) to hide the visualization menu when we're not viewing the visualization. That's up to you! This is where setting a menu token can be useful, since if we ever decide to change the order or this menu or add new stuff, we don't have to update any disable/hide when rules if our menu indices changed.

Next, we'll add a parameter callback to this new menu that will switch the **Output for View Flag** for us when we change it.

### Quick Recap: What is a parameter callback?

Whenever a user changes a parameter in the interface (or runs a
`hou.Parm.pressButton()` with Python), Houdini runs whatever code is in the **Callback Script** field in the parameter's **Parameter Description** from the **Type Properties** or **Edit Parameter Interface** window.

{{< figure src="images/callback-script-location.png" title="" caption="Callback script location" alt="Callback script location" >}}

{{< video src="images/callback-demo.mp4" >}}

> See the SideFX docs on [Parameter Callback Scripts](https://www.sidefx.com/docs/houdini/hom/locations.html#parameter_callback_scripts) for more info.

---

### Writing our Callback

When writing parameter callback scripts in Python, we get access to a `dict` called `kwargs` that passes some super useful info about the parameter and its state.

Let's print the value of `kwargs` and see what's in it:

{{< figure src="images/kwargs-dict-contents.png" title="" caption="`kwargs` `dict` contents" alt="`kwargs` `dict` contents" >}}

```python
{
    'node': <hou.SopNode of type subnet at /obj/geo4/fracture>,
    'parm': <hou.Parm display in /obj/geo4/fracture>,
    'script_multiparm_index': '-1',
    'script_value0': 'viz',
    'script_value': 'viz',
    'parm_name': 'display',
    'script_multiparm_nesting': '0',
    'script_parm': 'display'
}
```

One key in particular stands out: `script_value`. This is the new value that the parameter was just set to. One gotcha with this is that in this dictionary, it always returns a `string`, and in this case it's the token string we added when we created the menu. We should keep this in mind when we write the callback.

{{< figure src="images/menu-token-labels.png" title="" caption="Menu token/labels" alt="Menu token/labels" >}}

We can use one of the following snippets for our callback:

```python
# In our case, we only have two menu items (0, 1) and two outputs
kwargs["node"].setOutputForViewFlag(kwargs["parm"].evalAsInt())
```

or if we'd rather compare the string from the menu token:

```python
kwargs["node"].setOutputForViewFlag(kwargs["script_value"] == "viz")
```

Now whenever we switch this menu, the **Output for View Flag** is updated on our node, and we can switch between seeing the viz geo and our actual output!

{{< video src="images/switching-the-menu.mp4" >}}

### One Step Further

This is pretty good, but if we wanted we could take it another step further and instead of using two menus, flatten it down into one. We're kind of doing a mix of the [first method](#simple-switch-sop-and-an-ordered-menu-parameter) and the [additional output method](#additional-output) above.

{{< figure src="images/single-menu.png" title="" caption="Single Menu" alt="Single Menu" >}}

Let's get rid of the second menu, and add an item called "Fractured Pieces" to index 0 of the **Display** menu. Now the menu has all the items in it, a lot like we had in the first example method.

{{< figure src="images/update-index-zero.png" title="" caption="Update index zero" alt="Update index zero" >}}

#### Update the Switch

Since our menu has one more element in it (the "final" output selection in index 0), we need to subtract `1` from the switch lookup.

{{< figure src="images/update-the-switch-index.png" title="" caption="Update the switch index" alt="Update the switch index" >}}

#### Update the Callback

Finally, we'll add a callback for this menu:

{{< figure src="images/updated-callback.png" title="" caption="Updated Callback" alt="Updated Callback" >}}

```python
kwargs["node"].setOutputForViewFlag(kwargs["script_value"] != "output")
```

or

```python
kwargs["node"].setOutputForViewFlag(kwargs["parm"].evalAsInt() > 0)
```

And that's it! We can switch between our visualization geometry / intermediate step stuff from a single menu, but leave our actual output untouched!

{{< video src="images/output-for-view-single-menu.mp4" >}}

# Final Thoughts

## Housekeeping

- Label your outputs!

    The **Output for View Flag** label in the Network Editor gets its name from the name of the **Output SOP**.
    {{< figure src="images/output-for-view-flag-label.png" title="" caption="Output for View Flag Label" alt="Output for View Flag Label" >}}

    When you hover over an output port of an **HDA**, that label is set from the **Type Properties** dialog. Also, when this is set, the **Output for View Flag** label in the Network Editor will use this instead of the Output node's name.

    {{< figure src="images/output-label-type-properties.png" title="" caption="Output Label (Type Properties)" alt="Output Label (Type Properties)" >}}

- Color your inputs/outputs if you like

    {{< figure src="images/input-output-colors.png" title="" caption="Input and output ports take on the same color as their corresponding nodes inside" alt="Input/Output Colors" >}}

---

## Switch SOPs and Bitfields from Button / Icon Strips

Another post coming soon!
