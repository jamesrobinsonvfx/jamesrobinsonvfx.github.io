---
title: "opchange"
date: 2021-03-26
draft: false
ShowReadingTime: true
summary: "Quickly Search/Replace All Parameters in Your Hipfile"
taxonomies:
    categories: tips workflow
    tags: hscript expressions parameters variables
cover:
    image: images/preview.png
    alt: "opchange"
    caption: "`opchange` example"
    relative: true
---

**Easily switch any value for another in a hipfile.**


Going through and manually swapping `$HIP` for `$JOB`, or switching any value for another noe in an entire hipfile can be a real pain. You could try going the long way (looking at each parm and changing by hand...ouch!), or by iterating all the parms on all the nodes with Python. Or, you can use a simple built-in HScript command: `opchange`.

[`opchange`](https://www.sidefx.com/docs/houdini/commands/opchange.html) makes it really simple to swap values across the entire scene,  or over a limited scope of nodes.

## Example
Let's say you were loading a bunch of HDRIs from some sort of library you have elsewhere on disk, but now you need to share the project with someone else. You've copied all the light textures from you library location into `$JOB/tex`, but now you need to change all the paths on the lights in your hipfile. We can use `opchange` to make it a bit easier.

```shell
opchange D:/Library/HDRI \$JOB/tex
```

Now all references to `D:/Library/HDRI` should have been replaced with `$JOB/tex`.

## Gotchas
If you're swapping a variable, and you want to search for the *unexpanded* version, ie the literal string `$HIP`, make sure to escape the `$` with a `\`, otherwise Houdini will be searching for the *expanded* versions, like `~/project/myproject` or wherever `$HIP` is pointing to instead.

```shell
opchange \$HIP \$JOB/hip
```

### Limit the Scope
We don't have to search the entire scene in this case, since we know that all of our lights are going to exist at `/obj`.

```shell
opchange -p /obj D:/Library/HDRI \$JOB/tex
```

Duplicated a node and want to change the new node's parm reference? Use the `-i` flag.

```shell
opchange -i -p /obj/sheepnmoon/mat/fuzz_moon sheep_ moon_
```