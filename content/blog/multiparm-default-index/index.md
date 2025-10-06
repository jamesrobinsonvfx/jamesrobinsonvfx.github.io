---
title: "Multiparm Default Expressions"
date: 2025-10-02T12:42:34+13:00
draft: true
ShowReadingTime: true
ShowToc: true
summary: "Set the default value of a parameter in a multiparm so that it always references another parameter in the same multiparm instance."
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Cover Photo"
    relative: true
categories: ["houdini"]
tags: ["tip", "multiparm"]
---

*Written for H21, but applies to previous versions of Houdini as well.*

I often need to use multiparms when creating tools, and a question that comes up from time to time is:

> *How do I set the default value of a parameter in a multiparm that references another parameter in the same multiparm instance?*

Why would you want to do that anyway?
* Your parameter value needs to be the result of combining some other related parameters.
* You want to have a unique label for each multiparm based on some other parameter values.

Let’s say we have a multiparm interface like this one, and we want the **Voxel Size** parameter in each multiparm instance to always default to being the product of each **Grid Scale** and **Particle Separation** parameter:

{{< figure src="images/scenario.png" title="" caption="Example scenario" alt="Example scenario" >}}

At first it might seem easy enough, but inserting and deleting multiparm instances in previous versions of Houdini would often cause issues where channel references that were set as default values would break or point to an unexpected parameter after inserting/deleting. Now that Houdini 21 gives us the ability to easily shuffle multiparms around, this problem could start cropping up more, so let’s check out how we can make sure our defaults are stable.

## But first, Some Quick Facts About Multiparms
[Houdini Docs: Multiparm Block](https://www.sidefx.com/docs/houdini/ref/windows/optype.html#multiparm-block)
* Each time you increment/decrement the multiparm number, you create/remove a multiparm ***instance***.
* Houdini adds a number or ***index*** to each instance.
* In the Parameter Editor, multiparms are templates, and the parm name itself has a hash in it which gets replaced by the multiparm instance index.
* Nested multiparms use multiple hash tokens to denote their level. Something like `#_#` for the third element in the second multi-parm would become `2_3`.
* Prior to H21, multiparms could not be re-ordered. Only insert, append, and remove operations were supported.
* Hash tokens `#` can be used in default parameter expressions, and the default value for the parameter instance will have the index hard-coded in its default value (more on that below).


## Problem Overview

> *How can we set the default value for the **Voxel Size** parameter so that it always references the **Particle Separation **and** Grid Size** parameters from the same multiparm?*

Let’s keep using the example above to explore some options.

## First Attempt: Hash # Token

Parameters inside multiparm blocks are required to have a `#` in their name (if you don't add one, it will be added for you automatically). This `#` character can be used elsewhere too, like in the **Disable When / Hide When Condition** and the **Default Value**.

### Literal Default

You can use the `#` token as the default value for a parameter in a multiparm block. This will **hardcode the multiparm index into that parameter instance's default value**

{{< figure src="images/simple-literal-demo.png" title="" caption="`#` token literal default" alt="`#` token literal default" >}}

{{< figure src="images/simple-literal-demo-parms.png" title="" caption="`#` as the parameter default. Notice how none of the parameters are bold, meaning they are all at their default value." alt="`#` as the parameter default. Notice how none of the parameters are bold, meaning they are all at their default value." >}}


### Hscript Default

Hopping back to our original example, we can use the `#` token to create default channel references.

{{< figure src="images/hash-token-hscript-setup.png" title="" caption="`#` token Hscript setup" alt="`#` token Hscript setup" >}}

1. Select the parameter.
2. Go to the **Channels** tab.
3. Change the **Default Value's Expression Language** to **Hscript**.
4. Set the expression using the `#` token in the parameter names.
   ```c
   ch("psep#") * ch("gridscale#")
   ```
{{< figure src="images/hash-token-hscript-result.png" title="" caption="Each **Voxel Size** parameter looks correct and references the parameters from the same instance!" alt="Each **Voxel Size** parameter looks correct and references the parameters from the same instance!" >}}

Though if you look at the default expression for each parameter, you'll see that the `#` token has been replaced by the multiparm instance's index:

{{< figure src="images/hash-token-hscript-expressions.png" title="" caption="`#` token has been replaced..." alt="`#` token has been replaced..." >}}

```c
ch("psep2") * ch("gridscale2")
```

At first this might seem like no biggie. And you may never notice it if all you ever do is add/remove from the bottom of the multiparm block using the **[+,-,Clear]** buttons.

But what happens if we start to reorder these? Let's move **Sim 3** to **Sim 1**'s position:

{{< figure src="images/hash-token-hscript-reorder.png" title="" caption="Drag **Sim 3** to **Sim 1**" alt="Drag **Sim 3** to **Sim 1**" >}}

{{< figure src="images/hash-token-hscript-reorder-incorrect.png" title="" caption="Something's not right..." alt="Something's not right..." >}}

Well that's not right now is it? You would expect the value of **Voxel Size** to stay the same as it was when it was at the bottom before reordering. Now it appears to be saying that `0.05 * 1.0 = 1.0`? Also, all of the **Voxel Size** parameters for every other parameter have now gone bold, meaning they're no longer at their default value. So what's going on?

If we take a look at the expressions we'll see that they kept their old values, so the channel references are pointing to the wrong parameters now:

{{< figure src="images/hash-token-hscript-reorder-expressions.png" title="" caption="Channel refs are still using their old index." alt="Channel refs are still using their old index." >}}

One solution would be to revert each **Voxel Size** parameter to its default:

{{< figure src="images/hash-token-hscript-reorder-revert.png" title="" caption="**Right Click → Revert to Defaults**" alt="**Right Click → Revert to Defaults**" >}}

But this is pretty unreasonable to ask people to do, especially when there are many parameters. There must be a better way!

#### Pros & Cons
* ✅ Can be useful for simple cases where you want the value to match the index number.
* ✅ Hardcodes the index digit into the parameter's default value.
* ✅ Easy to set up for nested multiparms.
* ❌ Hardcodes the index digit into the parameter's default value.
* ❌ Can lead to unexpected behavior when the order of the instances in the multiparm block changes, since the index the instance was created as is hardcoded into the default value and ***does not update*** when re-ordered.

## Hscript Solution: Use `$CH`

Since the `#` character gets swapped out and hardcoded to the multiparm index number, we can't rely on it for relative references when reordering parameters since the parameter name **does** update and get a new index number.

So we need something a bit more flexible. We need to be able to to look at the current parameter and get its multiparm index. This is straightforward in Python, but before we start jumping there let's see if there's a simpler way with Hscript. Since the index is part of the parameter's name, maybe we should look there first. But how can we get the parameter's name?

AFAIK there is no Hscript function like `opname` for parameters - but there is a variable `$CH` which will return the current channel name!

> [Houdini Docs: $CH](https://www.sidefx.com/docs/houdini/network/expressions.html#channels)

{{< figure src="images/ch-raw.png" title="" caption="" alt="`$CH` expression in a string parm" >}}
{{< figure src="images/ch-expanded.png" title="" caption="`$CH` variable evaluates to the current channel (parm) name." alt="`$CH` variable evaluates to the current channel (parm) name." >}}

### `opdigits` and `$CH`

Now that we have the parameter name, we need to extract the index number from it. This is a perfect job for our friend `opdigits()`.

> [Houdini Docs: `opdigits`](https://www.sidefx.com/docs/houdini/expressions/opdigits.html)
>
> “…*return the numeric value of the **last set of consecutive digits** in a node's<sup>1</sup> name*”
> >
> <sup>1</sup> <small>Despite the wording in the docs, you can use this on **any** string, not just node paths.</small>

We can use this to quickly grab the index from the parameter name. Since our multiparms have the `#` token at the end and no other digits after it will work just fine:
```c
opdigits($CH)
```

{{< figure src="images/opdigits-ch-expression.png" title="" caption="" alt="`opdigits($CH)` expression" >}}
{{< figure src="images/opdigits-ch-eval.png" title="" caption="`opdigits($CH)` for a parameter called `int_31415`" alt="`opdigits($CH)` for a parameter called `int_31415`" >}}

### Solution

Let's apply this to our multiparm example from earlier:

{{< figure src="images/opdigits-ch-multiparm-setup.png" title="" caption="Edit Parameter Interface" alt="Edit Parameter Interface" >}}

1. Select the parameter.
2. Head to the **Channels** tab.
3. Set the **Default Expression Language** to **Hscript**.
4. Set the default expression using `opdigits($CH)` instead of the `#` token:
```c
ch("psep" + opdigits($CH)) * ch("gridscale" + opdigits($CH))
```
or
```c
{
    string i = opdigits($CH);
    return ch("psep" + i) * ch("gridscale" + i);
}
```

{{< figure src="images/opdigits-ch-multiparm-expressions.png" title="" caption="New default expression using `opdigits($CH)` instead of `#`." alt="New default expression using `opdigits($CH)` instead of `#`." >}}

{{< figure src="images/opdigits-ch-multiparm-eval.png" title="" caption="Updated expression with `opdigits($CH)` evaluated." alt="Updated expression with `opdigits($CH)` evaluated." >}}

{{< figure src="images/opdigits-ch-multiparm-reorder.png" title="" caption="What happens if we move **Sim 3** to **Sim 1**'s spot this time? Everything is still correct after reordering!" alt="What happens if we move **Sim 3** to **Sim 1**'s spot this time? Everything is still correct after reordering!" >}}

#### Pros & Cons
* ✅ Index is not hardcoded. Works no matter if you reorder/append/delete multiparm instances.
* ✅ Easy and quick with Hscript, no Python required.
* ❌ Does not support nested multiparms well.
* ❌ Depending on the parm name, this could break depending on where the hash character is in the name if there are numbers in the parm name already, either from a user or a parm with size>1 (see below).
* ❌ Can be cumbersome to write Hscript sometimes.

## Limitations of `opdigits($CH)`

While the `opdigits($CH)` solution is quick to implement and works pretty well in a lot of cases, it does have a few drawbacks.

### ❌ Numbers in Parameter Names

Sometimes parameters will already have digits in their names. This could cause a problem depending on where the digit is in the name.

#### User-created Parameter Names

When the digit is part of your own naming scheme, it's usually easier for you to just change it. If you absolutely must have a digit in the name, try to to make sure the `#` token is at the **end** of the parameter name if possible:

{{< figure src="images/parmname-user-digit-end.png" title="" caption="❌ Don't put your digit after the `#` token if you can help it." alt="❌ Don't put your digit after the `#` token if you can help it." >}}

{{< figure src="images/parmname-hash-token-end.png" title="" caption="✅ Keep the hash at the end if you can." alt="✅ Keep the hash at the end if you can." >}}

#### Float & Integer Parameters With Size > 1

It gets trickier with **Float** or **Integer** parameters that have a ***size greater than 1***, because Houdini automatically appends a digit to the end of each one:

{{< figure src="images/float-size-3.png" title="" caption="Houdini adds a digit for each parameter inside the parmtuple." alt="Houdini adds a digit for each parameter inside the parmtuple." >}}

One possible workaround could be to instead use the **Float Vector** or **Integer Vector** parameter types instead if that suits you (assuming the size of the parameter is 2, 3, or 4).That way the parameters will get `x,y,z,w` added to the end of the name instead of `1,2,3,4`.

{{< figure src="images/vector-parms.png" title="" caption="Float/Integer Vector Parameter Types" alt="Float/Integer Vector Parameter Types" >}}

{{< figure src="images/float-vector-parmtype.png" title="" caption="`1,2,3` becomes `x,y,z`" alt="`1,2,3` becomes `x,y,z`" >}}


### ❌ Nested Multiparms

Remember that `opdigits` only gives us the ***last consecutive digits*** of a string. When **nesting** multiparm blocks inside of other multiparm blocks, you need to add another `#` token to the parameter name for each level of nesting. Most commonly this is done by adding an underscore and another `#`:
```c
myparameter#_#
```

In this case `opdigits` won't give us enough info.

## Nested Multiparms

As mentioned above, our `opdigits` trick breaks down once we start nesting multiparms.

Let's update our scenario to see how we can handle nested multiparms. We'll create an interface with one multiparm block that populates a list of sequences, and inside each sequence there is another multiparm block that populates shots.

{{< figure src="images/nested-example-start.png" title="" caption="Nested multiparm example." alt="Nested multiparm example." >}}

In this case, let's make sure that the default for **Shot Name** is always:
```
Sequence Name_Shot Number
```

### Hash Token Revisited

Take a look at the parameter names for the nested multiparm. Another `_#` is added for each level of nesting:

{{< figure src="images/nested-example-setup.png" title="" caption="Notice the double hash token for the nested multiparm `#_#`." alt="Notice the double hash token for the nested multiparm `#_#`." >}}

We can try to do the same thing as we did before, only this time we'll add the extra `#` to our channel reference:

{{< figure src="images/nested-example-hash-hscript-setup.png" title="" caption="Nested multiparm Hscript default using `#_#` token." alt="Nested multiparm Hscript default using `#_#` token." >}}

```c
chs("seq#") + "_" + chs("shot#_#")
```

{{< figure src="images/nested-example-hscript-eval.png" title="" caption="`#_#` evaluated inside an Hscript default expression" alt="`#_#` evaluated inside an Hscript default expression" >}}

{{< figure src="images/nested-example-hash-hscript-backwards.png" title="" caption="The numbers are backwards! This feels like a bug 🐛." alt="The numbers are backwards! This feels like a bug 🐛." >}}

The first instance looks correct, but it seems like the order of the `#` tokens gets messed up as we add more. My guess is that since we used the `#` token already in `chs("seq#)`, that by the time it gets to `chs("shot#_#")` the parser sees that we've used the 2 `#` tokens already instead of treating each `chs()` section separately. Switching the expression to a string literal and using backticks doesn't help either, so we might be out of luck for this case!

### 🐍 Python to the Rescue

When all else fails or you have a particularly interesting scenario, you can always use a Python expression. `hou.Parm` has several methods for dealing with multiparms, the most useful one in our case being `hou.Parm.multiParmInstanceIndices`.

> Houdini 21 added a few more multiparm methods. Check 'em out!
>
> [Houdini Docs: HOM Multiparms](https://www.sidefx.com/docs/houdini/hom/hou/Parm.html#multiparms)

#### Solution: Using `hou.Parm.multiParmInstanceIndices` to Find the Right Index

{{< figure src="images/nested-example-python-eval.png" title="" caption="Default Python Expression for the Shot Name parameter returns the correct result now." alt="Default Python Expression for the Shot Name parameter returns the correct result now." >}}

Here's the final setup - I'll step through the code in more detail below. Our new default expression looks something like this:

```python
i, j = evaluatingParm().multiParmInstanceIndices()
seq = evalParm(f"seq{i}")
shot = evalParm(f"shot{i}_{j}")
return f"{seq}_{shot}"
```

or shorter with nested f-strings (some might say this is a bit less readable):

```python
i, j = evaluatingParm().multiParmInstanceIndices()
return f"{evalParm(f'seq{i}')}_{evalParm(f'shot{i}_{j}')}"
```

Don't forget to change the language of the default expression:

{{< figure src="images/nested-example-python-setup.png" title="" caption="Set the **Default Expression Language** to **Python** 🐍." alt="Set the **Default Expression Language** to **Python** 🐍." >}}

#### Explanation

First we need to get a reference to the current parameter object which we can do with `hou.evaluatingParm()`:

{{< figure src="images/evaluatingParm-example.png" title="" caption="Use `evaluatingParm()` to get the `hou.Parm` object for the parameter." alt="Use `evaluatingParm()` to get the `hou.Parm` object for the parameter. You can leave the `hou.` out of Python parameter expressions." >}}

```python
evaluatingParm().multiParmInstanceIndices
```

Then we use `hou.Parm.multiParmInstanceIndices` to guarantee that we're getting the correct multiparm index for our parameter. This function returns a tuple with all of the multiparm indices from top to bottom, similar to what you would get from the `#` token:

{{< figure src="images/multiParmInstanceIndices-example.png" title="" caption="`multiParmInstanceIndices()` in action." alt="`multiParmInstanceIndices()` in action." >}}

Once we have that we just need to return the value! In this case we grab the corresponding **Sequence Name** and **Shot Number**:

```python
seq = evalParm(f"seq{i}")
shot = evalParm(f"shot{i}_{j}")
```

and squish them together:

```python
return f"{seq}_{shot}"
```

#### Pros & Cons
* ✅ Can handle more scenarios.
* ✅ More flexible than Hscript.
* ❌ Requires a Python expression and knowledge of HOM.
* ❌ Needs a bit more care in exception handling to not be annoying depending on what you’re doing.

## Conclusion

And there you have it! A few methods you can use for setting default values in multiparms that consistently refer to another multiparm regardless of reordering. I like to keep it simple and stick with Hscript as long as I can, but in the end you should be able to solve most challenges with a Python expression if you really need to.

Thanks for reading!
