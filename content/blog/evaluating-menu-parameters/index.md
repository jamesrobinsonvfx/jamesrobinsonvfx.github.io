---
title: "Evaluating Menu Parameters"
date: 2023-01-30
draft: false
ShowReadingTime: true
ShowToc: true
summary: "Explore the results of evaluating menu parameters with various methods"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Evaluating menu parameters"
    relative: true
categories: ["houdini"]
tags: ["reference", "ui", "houdini", "parameters", "python", "menu"]
---

# Menu Parameters & Evaluation Results

I can never remember off the top of my head when a menu parameter is going to evaluate the token string, or an integer, or a string representation of an integer without first doing a quick little check in the Python Shell...so here's a table that should help future James and maybe some other folks as well. Each test was done with the parameters set to use a [Normal](#normal-menu) menu, with the tokens `item0`, `item1`...

| Parameter Type | `hou.Parm.eval()` | `hou.Parm.evalAsString()` | `hou.Parm.evalAsInt()` | `ch()` | `chs()` | `kwargs["script_value"]` |
|----------------|-------------------|---------------------------|------------------------|--------|---------|--------------------------|
| Ordered Menu   | `1`               | "item1"                   | `1`                    | `1`    | "item1" | "item1"                  |
| String         | "item1"           | "item1"                   | `TypeError`            | `0`    | "item1" | "item1"                  |
| Integer        | `1`               | "1"                       | `1`                    | `1`    | "1"     | "1"                      |

> Cells with formatting like `0`, and `1` are integer types. Cells in double quotes `"` are string types.

# Overview

Houdini offers a few ways to both create and evaluate dropdown menu parameters.

The way you configure these menu parameters affect how they get evaluated in different contexts (hscript, Python, parameter callbacks, etc.). Let's take a look at what that means, and come up with a quick reference table that we can use when debugging or creating new interfaces.

> Hipfile: [jamesr_evaluatingmenuparameters.hiplc](houdini/hip/jamesr_evaluatingmenuparameters.hiplc)

# Menu Parameter Types

{{< figure src="images/typical-dropdown-menu.png" title="" caption="Typical dropdown menu" alt="Typical dropdown menu" >}}

## What's a menu made of?

Houdini menus are made up of *item/label pairs*.

{{< figure src="images/item-label-pairs.png" title="" caption="Item/Label Pairs" alt="Item/Label Pairs" >}}

### Label

Labels are the "nice names" that the user sees in the dropdown.

Using Python, you can get a list of a menu parameter's labels with

```python
hou.parm("/obj/geo1/null1/greetings").menuLabels()

('Hi', 'Hallo', 'Kia ora', 'Hej hej', 'こんにちは')
```

> If you ever want to get the label of the currently selected menu item, try this:
>```python
>parm = hou.parm("/obj/geo1/null1/greetings")
>parm.menuLabels()[parm.evalAsInt()]
>
>'Hallo'
>```

### Item (Token)

A *Menu Item* is the internal name used by Houdini. You can optionally specify a *token* for each menu item which can be used for setting and evaluating the menu as a string, and for use with [disable/hide when rules](https://www.sidefx.com/docs/houdini/ref/windows/optype.html#conditions).

Just because you put something as the token, doesn't mean that's *always* what you'll get when you evaluate it ([see the table above](#menu-parameters--evaluation-results)).

Using Python, you can get a list of a menu parameter's items/tokens using:

```python
hou.parm("/obj/geo1/null1/greetings").menuItems()

('hi', 'hallo', 'kiaora', 'hej', 'konnichiwa')
```

## Ordered Menu

In general, most of the menu parameters I come across (and create) use the **Ordered Menu** parameter type. This parameter type creates the classic dropdown that you've likely seen countless times:

{{< figure src="images/cover.png" title="" caption="Typical dropdown menu" alt="Typical dropdown menu" >}}

{{< figure src="images/ordered-menu-parm-type.png" title="" caption="Typical dropdown menu settings" alt="Typical dropdown menu settings" >}}

These can be created directly in the Type Properties / Edit Parameter Interface UI by either explicitly specifying a list of token/label pairs

{{< figure src="images/menu-item-label-pairs-only.png" title="" caption="Configuring and Ordered Menu parameter in the Edit Parameter Interface window" alt="Configuring and Ordered Menu parameter in the Edit Parameter Interface window" >}}

or more dynamically through a Python parameter menu script.

{{< figure src="images/python-menu-script-adv.png" title="" caption="An example of Python Parameter Menu Script" alt="An example of Python Parameter Menu Script" >}}

{{< collapse summary="`greetings.json`" >}}
[`greetings.json`](greetings.json)
```json
{
    "greetings": [
        {"language": "english", "token": "hi", "label": "Hi"},
        {"language": "german", "token": "hallo", "label": "Hallo"},
        {"language": "te reo", "token": "kiaora", "label": "Kia ora"},
        {"language": "swedish", "token": "hej", "label": "Hej hej"},
        {"language": "japanese", "token": "konnichiwa", "label": "こんにちは"}
    ]
}
```
{{< /collapse >}}

> For more info on generating menus with Python:
>
> [Generating a menu from attributes](https://www.sidefx.com/docs/houdini/hom/hou/SopNode.html#generateInputAttribMenu)
>
> [Parameter Menu Scripts](https://www.sidefx.com/docs/houdini/hom/locations.html#parameter_menu_scripts)
>
> [Quick Menu Script List](https://gist.github.com/jamesrobinsonvfx/226a72a591e7663ea1fade594c1e13a4)

## String or Integer Parameter with "Use Menu" Enabled

Another common place to use menus are **String** and **Integer** parameter types.

These menus are often used to help populate a field, for instance when selecting several attributes to get rid of in the **Attribute Delete SOP**.

{{< figure src="images/attribdelete-mini-menu.png" title="" caption="Attribute Delete SOP Menu with attributes expanded" alt="Attribute Delete SOP Menu with attributes expanded" >}}

A regular **String** parameter type can be made to use a menu by enabling the following toggle on the right side of the Type Properties / Edit Parameter Interface window:

{{< figure src="images/use-menu-toggle.png" title="" caption="Use Menu toggle" alt="Use Menu toggle" >}}

<a name="normal-menu"></a>
When using the **Normal** option, the menu will appear just like the ordinary **Ordered Menu** parameter type. The other two common options are **Replace** and **Toggle** - these will add a small mini-menu to the right of the string/int parameter from which you can select and populate the string/int field. Those are the styles that are most familiar from nodes like **Attribute Delete** etc.

### Why would I ever use the "Normal" mode if I'm using a string or int parameter? Can't I just use an Ordered Menu?

Usually an **Ordered Menu** is just fine - but one advantage is explicitness. If you want to make sure your menu always returns a `string` when using HOM's `hou.Parm.eval()`, you can create a **String** parameter, enable **Use Menu**, and keep it as **Normal (Menu Only, Single Selection)**. Whenever someone evaluates the parameter with `hou.evalParm("/obj/geo1/null1/mymenuparm")`, they *will* get a string back which is *not* the case with the standard **Ordered Menu** type (which would *need* `hou.Parm.evalAsString()`, even if your token *looks* like a string).

## Test Cases

For testing, I'm using the following menu parameters:

- Ordered Menu
- String Parameter with [Normal](#normal-menu) menu enabled
- Integer Parameter with [Normal](#normal-menu) menu enabled

{{< figure src="images/test-case-menus.png" title="" caption="Menus on the Parameter Interface" alt="Menus on the Parameter Interface" >}}

They each have different labels, but the menu items/tokens are all the same: `item0`, `item1`...

{{< video src="images/test-case-menu-items.mp4" >}}

The results in the tables below were gathered when the menus were changed to their second dropdown item (`item1`).

See the [hipfile](#hipfile) above to check it out yourself.

{{< figure src="images/menu-testing-interface.png" title="" caption="Menu testing interface from hipfile" alt="Menu testing interface from hipfile" >}}

<details markdown=1><summary markdown="span">`hou.Parm.eval()`</summary>

### `hou.Parm.eval()`

```python
value = parm("orderedmenu").eval()
return f"{value} {type(value)}"
```


| Parameter Type | `hou.Parm.eval()` | Type |
|----------------|-------------------|------|
| Ordered Menu   | 1                 | int  |
| String         | item1             | str  |
| Integer        | 1                 | int  |

</details>

<details markdown=1><summary markdown="span">`hou.Parm.evalAsString()`</summary>

### `hou.Parm.evalAsString()`

```python
value = parm("orderedmenu").evalAsString()
return f"{value} {type(value)}"
```

| Parameter Type | `hou.Parm.evalAsString()` | Type |
|----------------|---------------------------|------|
| Ordered Menu   | item1                     | str  |
| String         | item1                     | str  |
| Integer        | 1                         | str  |

</details>

<details markdown=1><summary markdown="span">`hou.Parm.evalAsInt()`</summary>

### `hou.Parm.evalAsInt()`

```python
value = parm("orderedmenu").evalAsInt()
return f"{value} {type(value)}"
```

| Parameter Type | `hou.Parm.evalAsInt()` | Type      |
|----------------|------------------------|-----------|
| Ordered Menu   | 1                      | int       |
| String         |                        | TypeError |
| Integer        | 1                      | int       |

</details>

<details markdown=1><summary markdown="span">`ch()`</summary>

### `ch()`

```
ch("orderedmenu")
```

| Parameter Type | `ch()` | Type |
|----------------|--------|------|
| Ordered Menu   | 1      | int  |
| String         | 0      | int  |
| Integer        | 1      | int  |

</details>

<details markdown=1><summary markdown="span">`chs()`</summary>

### `chs()`

```
chs("orderedmenu")
```

| Parameter Type | `chs()` | Type |
|----------------|---------|------|
| Ordered Menu   | item1   | str  |
| String         | item1   | str  |
| Integer        | 1       | str  |

</details>

<details markdown=1><summary markdown="span">`kwargs["script_value"]`</summary>

### `kwargs["script_value"]`

```python
value = kwargs["script_value"]
hou.ui.displayMessage(f"kwargs['script_value'] = {value}\n\n{type(value)}")
```

{{< figure src="images/test-callback.png" title="" caption="Parameter Callback (Python)" alt="Parameter Callback (Python)" >}}

| Parameter Type | `kwargs["script_value"]` | Type   |
|----------------|--------------------------|--------|
| Ordered Menu   | item1                    | str    |
| String         | item1                    | str    |
| Integer        | 1                        | str    |

When implementing parameter callbacks, Houdini gives us access to a Python dictionary named `kwargs` filled with useful info about the parameter. One of those keys is called `script_value`, which according to the docs is equivalent to `kwargs["parm"].eval()` (see above - this isn't quite always the case, especially when it comes to int/float parameters, since it always seems to return a string).

> See Also [Parameter Callback Scripts](https://www.sidefx.com/docs/houdini/hom/locations.html#parameter_callback_scripts)

</details>

# Final Thoughts

**Ordered Menus** and **String** Parameters can be ***set*** using their tokens like so:

```python
hou.parm("/obj/geo1/null1/orderedmenu").set("item1")
```
---

When an integer parm using a [Normal](#normal-menu) menu that has tokens is evaluated as a string, it will not return the token, but rather a string representation of the selected index.

You *cannot* set an integer menu parameter by its token like you can with a string parameter / ordered menu ***unless the token you're trying to set it to is a string representation of a numerical value, like `"1000"`, `"42"`*** etc.

---

## Use Token as Value

[This reply](https://www.sidefx.com/forum/topic/82576/?page=1#post-354880) from SESI on the SideFX Forums describes well what **Use Token as Value** does.

When you have **Use Token as Value** enabled on an integer parameter with a menu (or an Ordered Menu), and your tokens are string representations of numbers like `"1000"` or`"42"`, instead of evaluating to the *menu index* as usual, Houdini will try to use that value instead.

For instance, if our integer parameter has the following menu:

| Token | Label       |
|-------|-------------|
| item0 | Int Label 0 |
| item1 | Int Label 1 |
| 42    | Forty Two   |

and we have **Use Token as Value** *disabled*, calls like `hou.Parm.eval()` and `ch()` will evaluate to the selected menu index `2`

{{< figure src="images/use-token-as-value-disabled.png" title="" caption="Use Token and Value disabled" alt="Use Token and Value disabled" >}}

Once we turn on **Use Token as Value**, we get `42`

{{< figure src="images/use-token-as-value-enabled.png" title="" caption="Use Token as Value enabled" alt="Use Token as Value enabled" >}}

See it in use for real on the **Vellum Constraints** node's `stretchstiffnessexp` parameter.
