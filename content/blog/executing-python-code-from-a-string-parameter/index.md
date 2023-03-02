---
title: "Executing Python Code From a String Parameter"
date: 2022-02-18
draft: false
ShowReadingTime: true
summary: "Create a preset on a Null that lets you run Python code from it"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Execute Python code from a string parameter"
    relative: true
categories: ["houdini"]
tags: ["tips", "ui", "python"]
ShowToc: true
hipfile: houdini/hip/jamesr_executingpythonfromstring.hiplc
---

{{< attachments >}}

# Overview

Sometimes you just want to run some python code, but don't want to run line-by-line in the Python Shell, can't be bothered to create a shelf tool, and don't need to run over geometry (so the Python SOP is out). Luckily, it's really easy create a simple preset that lets us run some Python code from a string parameter on any node.

This is useful for creating small scripts to run in your scene, debug stuff,
prototype some code, and even for creating code for parameter callbacks on the fly without needing to write your code to disk, or embed into an HDA.

> This method uses Python's `exec` function, and can be used to run any arbitrary string as code. Use with caution...

# Setup

Start off by creating a Null Object. This is where we'll add our parameters.

{{< figure src="images/01_null_sop.png" title="" caption="Null SOP" alt="Null SOP" >}}

Make all the current parms invisible by selecting them, and toggling the **Invisible** checkbox. We don't need to see them.

{{< figure src="images/02_make_invisible.png" title="" caption="Hide existing parameters" alt="Hide existing parameters" >}}

Add our own parameters and give them nice names and labels. We only need two:

* **Label**: *Execute* **Name**: *execute* **Type**: *Button*
* **Label**: *Code* **Name**: *code* **Type**: *String*

{{< figure src="images/03_add_parameters.png" title="" caption="Hide existing parameters" alt="Hide existing parameters" >}}

Currently, the interface is pretty basic and not something that would be very comfortable writing code in!

{{< figure src="images/04_basic_parms.png" title="" caption="Basic parameters" alt="Basic parameters" >}}

We can make the string parameter a bit roomier by modifying its Parameter Template here:

{{< figure src="images/05_modify_string_parm.png" title="" caption="Modify string parameter" alt="Modify string parameter" >}}

1. Enable *Multi-line String*
2. Set the Language to *Python*

Setting the language to Python doesn't really do anything special, aside from giving us some nice syntax hints and highlighting in the string input area.

{{< figure src="images/06_python_string_parm.png" title="" caption="Python string parameter" alt="Python string parameter" >}}

Nice, looking a lot better! Now we just need to hook it all up so that when we press the button, the code executes. We do this by adding a *Callback* to the button we created.

The callback code for this one is simple:

```python
exec(kwargs["node"].parm("code").evalAsString())
```

{{< figure src="images/07_add_callback.png" title="" caption="Add callback" alt="Add callback" >}}

> Make sure to set the callback language to Python if it isn't set already! You can tell by the icon to the right of the *Callback Script* parameter.

And that's it! Pressing the **Execute** button works just as a expected and prints our message to the shell.

{{< figure src="images/08_shell_output.png" title="" caption="Shell output" alt="Shell output" >}}

To finish it off, we can store this as a preset so that it's super quick to set up when we need it:

{{< figure src="images/11_save_preset.png" title="" caption="Save preset" alt="Save preset" >}}

{{< figure src="images/12_save_preset_dialog.png" title="" caption="Save preset dialog" alt="Save preset dialog" >}}

### A Quick Detour: `kwargs`

Now `kwargs` is a whole topic in itself (see the [documentation](https://www.sidefx.com/docs/houdini/hom/locations.html))! Basically, whenever you're running code on a node in Houdini, whether through a callback, an Action Button, a menu script, an `OnCreated` script ... and so on, Houdini usually passes you a really useful object called `kwargs`. `kwargs` generally stands for *Keyword Arguments*, and is a simple Python dictionary with some key/value pairs that can be really useful to you. I don't want to make this a post about kwargs, but just so we can understand it a little better, let's quickly erase the callback code that's currently in there, and replace it with:

```python
print(kwargs)
```

Checking the python shell, we can see that `kwargs` gives us access to the following info:

```python
{'node': <hou.ObjNode of type null at /obj/null1>, 'parm': <hou.Parm execute in /obj/null1>, 'script_multiparm_index': '-1', 'script_value0': '0', 'script_value': '0', 'parm_name': 'execute', 'script_multiparm_nesting': '0', 'script_parm': 'execute'}
```

so when we called `kwargs["node"]` above, we used the dictionary Houdini gave us for free to find the node that the parameter who was running the callback belongs to. This is generally a lot more reliable than using other methods such as `hou.pwd()` to find the current node. So use `kwargs` if it's available!

### Advanced: Complex Callbacks on the Fly

Now that we know how to run code on a node, and have some idea of what we get from `kwargs`, it's possible for us to actually write some more complicated code in a string parameter, and use it to run a callback that does something else.

The interesting part about this is that when we execute our code with the snippet above:

```python
exec(kwargs["node"].parm("code").evalAsString())
```

the `kwargs` dictionary is made available to us in our code block!

{{< figure src="images/09_kwargs_in_parm.png" title="" caption="`kwargs` in parm" alt="`kwargs` in parm" >}}

If we look closely, we can see that the `'parm_name'` key's value is `'execute'`, meaning that the `kwargs` dictionary was passed through from our button! This means we can write code in our string parameter and run it like a callback just like you can do on HDAs using the PythonModule script section! This can be extremely useful for creating scripts on the fly that run over multiple items in multiparm blocks. I'll include an example of that in the hipfile.

{{< figure src="images/10_advanced_mode.png" title="" caption="Advanced mode" alt="Advanced mode" >}}

Hopefully you enjoyed this, and thanks for reading!