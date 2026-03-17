---
publish: true
title: Blender
tags:
aliases:
graph:
  shape: box
  color: "#7f8c8d"
---
# Rendering Optimizations
Blender creates images by taking the 3D geometry and calculating how the light within the scene would interact with the geometry. Increasing the complexity of a scene, lighting, texture, or simulation will increase the amount of time necessary to get a finished photo. Animation's are ultimately just a series of photos so optimizing the per-image render time has a massive effect on the overall render time of a scene.

## Settings

#### GPU Render

This one is kind of obvious, GPU or GRAPHICS Processing unit. literally purpose built for this, make sure you're using it.

#### [Noise Threshold](https://ryankelly.garden/digital-garden/noise-threshold/)

Changing the Noise Threshold from 0.01 to 0.1 cuts the render time by nearly an order of magnitude. This is the threshold at which Blender stops rendering a scene and instead begins the De-noising Pass.

#### Light Paths

If you reduce the light paths to 1 and begin to increase them until you no longer see a difference, its a way to shave off a little bit of time. This will be sensitive to how many reflections are necessary.

#### Performance

Using Curves BVH, Disabling Compact BVH, and selecting Persistent Data can help speed up the render time a solid amount assuming nothing needs to move within your scene.

#### World Settings

Lowering the Map Resolution of the world map for HDRI's can help, but it can also make it worse

#### Instancing

Instead of Shift + D, use Alt + D. This instances the polygons which allows you to not have to render unique geometry but instances instead.

#### Culling

If the Camera can't see it, it might not need to be rendered.

#### Compression and Export Format

PNG's require compression on each frame, that's extra time on each frame. For animations, it likely makes sense to use OpenEXR format, Many people choose RGBA, Half color depth and DWAA CODEC

##### Sources

[5 Tips for FASTER Renders in Blender Cycles](https://youtu.be/VEdd9CynwQU?si=AzM_bFprjt2noQ9Q)  
[5 Blender Tips That Have Saved Me 100+ Hours](https://youtu.be/VoHZ8lemEtk?si=vIY7I9wQGyA3QU5w)

