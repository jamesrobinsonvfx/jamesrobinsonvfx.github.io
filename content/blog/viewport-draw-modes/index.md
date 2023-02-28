---
title: "Viewport Draw Modes"
date: 2021-03-26
draft: false
ShowReadingTime: true
summary: "Set different draw settings per viewport shading mode"
cover:
    image: images/preview.png
    alt: "Cover Photo"
    caption: "Viewport draw modes"
    relative: true
categories: [houdini, python]
tags: ["viewport", "optimization", "tip"]
---

I recently discovered that you can actually set the viewport drawing modes to be unique per Shading Mode. ie. when templating geo, you can shade with **Hidden Line Ghost** instead of the default template wireframe.

{{< figure src="images/settings.png" title="" caption="Viewport Settings" alt="Viewport Settings" >}}

This combined with a thinner wire width makes for a pretty pleasing experience! The ghost mode when templated also sort of helps show a bit more depth.
