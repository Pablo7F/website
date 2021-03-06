
+++
title = "Normal Map Hints"
description = "Lear how to use Normal Maps with Godot for 2D games."
author = "Pablo"

date = 2020-09-18T18:12:50-03:00
weight = 5
draft = true

difficulty = "beginner"
keywords = ["tutorial", "2D"]
+++

## What will you learn

*Normal maps* are textures used to increase the detail of dynamic lighting on surface.
With them, you can make a flat surface look like a complex geometry, with relief and
texture, when interacting with lights.

In this tutorial you will learn some common use cases of *normal maps* in Godot engin
for 2D games, and I will tell you some considerations you should know to make the
most of this powerful technique.

You can get the sprites to reproduce the following examples [here](https://github.com/GDQuest/godot-mini-tuts-demos/releases/download/untagged-1f11c44a177b76d8c6b0/NormalMapHintsAssets.zip).

## General Concepts

A *normal map* is a 2D texture in which each pixel color represents a *normal vector*.
A *normal vector* is a unit vector perpendicular to a surface. So when using a 
*normal map* on a surface, you are actually passing to the engine information about 
the geometry you would like that surface to have, and so the engine can make the 
calculations for dynamic lighting.

A deeper explanation about *normal maps* is out of the scope of this tutorial, but you
can refer to this sites if you want to learn more about them:

 * [What are normal maps? - Laigter´s documentation](https://laigter.readthedocs.io/en/latest/Introduction/intro.html#what-are-normal-maps)
 * [Normal mapping - Learn OpenGL](https://learnopengl.com/Advanced-Lighting/Normal-Mapping)
 
## Simple Sprite with *normal map* in Godot

Godot engine makes it really easy to use *normal maps* in 2D games. For this first
example, open Godot engine, create a new scene, and add a `Sprite` node to it. 
`Sprite` nodes have a "Normal Map" property that can be set in the *Inspector* dock.
You just need to drag an image from the *File System* dock, and drop it in the 
"Normal Map" property of the `Sprite`.

{{< video "videos/set-normal-map.mp4" "720" >}}

You won't be noticing anything, since normal maps only work with lights. So, go ahead
and add a `Light2D` node. Set a texture to it, and remember to increase the *Height*
property under the *Range* group in the inspector. If we assume all sprites are
placed in a plance, the light's height is how far the light is placed from that
plane, in the *Z* axis, and you need to increase it to make the lighting effect more
noticeable.

You should be able to notice now some dynamic lightng, that makes the sprite look 
like it has relief or depth. But the scene overall may be to bright and the effect 
is not as we would expect in more realistic lighting. For a better result, add a 
`CanvasModulate` node to the scene, and set its *Color* property in the inspector
to dark gray. Now the effect should look a lot better. `CanvasModulate` node lets
us, in this case, simulate *amient light*.

{{< video "videos/simple-sprite-with-normal.mp4" "720" >}}

## Seamless texture with *normal map*

Assume you have a seamless texture, that you want to use as background for your game.
For this purpose, you would enable *Repeate* and reimport the texture. The only 
thing you should note when using normal maps with such texture, is to also enable 
*Repeate* on the normal map, and reimport it.

Create a new scene, and add to it the same nodes that in the previous example 
(`Sprite`, `Light2D`, and `CanvasModulate`). Then, drag the texture and normal
map to the corresponding properties of the `Sprite` node. Also, enable *Region* 
property in the `Sprite`, and set the *Rect* to something bigger than the original
texture. As both the normal map and the texture are seamless, you should end up with
something like this:

{{< video "videos/seamless-with-normal.mp4" "720" >}}

Although this example may feel trivial after the first one, it lets us arrive to a 
conclusion that may not be obvious for someone not used to how things are rendered.
This is, the engine will use the same coordinates for the texture and normal map.
This will become relevant in the next few examples.

## Animations with Normal Maps

In the following examples, you will learn some tips about using *normal maps* with
animations.

There are two simple ways to create animations from a spritesheet in Godot engine. The first is
settings each frame in an `AnimationPlayer` node, and the second, using an `AnimatedSprite`.

{{< note >}}
Cutout animations are not covered in this tutorial, as it should be just a matter of
adding a *normal map* to each sprite.
{{< /note >}}

### Normal Maps with AnimationPlayer

Once again, create a new scene, with a `Sprite`, `Light2D`, and `CanvasModulate`
nodes. Add the spritesheet and its *normal map* to the corresponding properties of 
the `Sprite` node, and set the *Hframes* property, in the *Animation* group in the 
*Inspector*, to `11` (the number of frames our spritesheet has). Then, add an 
`AnimationPlayer` node, create a new animation called "Idle", with a duration of 
`0.8` seconds, loop enabled, and all the eleven frames. And that's all! As we
colncluded in the previous example, the engine will use the same coordinates for the
texture and the *normal map*, which meanse the corresponding region of the 
*normal map* will be used in each frame. You should see something like this:

{{< video "videos/animationplayer.mp4" "720" >}}

### Normal Maps with AnimatedSprite

Perhaps your animation is too simple to justify the use of an `AnimationPlayer`, 
and you would prefer to use an `AnimatedSprite` instead. But that node doesn't 
have a "Normal Map" property! However, with a really simple shader we can use 
*normal maps* with `AnimatedSprite`.

Add an `AnimatedSprite` node to the previous scene, and, in the *Inspector*, add
a new *SpriteFrames* in the *Frames* properties. Now you can create a new animation,
and click on the "Add frames from Spritesheet" button in the animation editor. In
the popup file dialog, select the spritesheet, and then set *Horizontal* and 
*Vertical* settings to `11` and `1` respectively. 
Finally, choose all eleven frames, and you should see the idle animation playing on 
your `AnimatedSprite`. For a better result, set the *FPS* setting in the animation
editor to `12`.

You can take a moment now, and compare how different the animation with and without 
normal map looks. Note how the animation with normal map looks as if the sprite had
volume.

{{< video "videos/comparing-animations.mp4" "720" >}}

Lets add a normal map. For this, got to the *Material* property of the
`AnimatedSprite` node in the *Inspector* and create a new *Shader Material*.
Create a new *Shader* in the corresponding property of the material, and add this
code to it:

```glsl
shader_type canvas_item;

uniform sampler2D normal_map;

void fragment(){
	NORMAL = 2.0*texture(normal_map, UV).rgb - 1.0;
}
```

The `normal_map` uniform defined in this shader will let you drag the normal map 
to the corresponding property under the *Shader Param* group in the inspector. 
In the `fragment()` function, we are sampling the normal map texture, 
transforming each sampled value to a valid unit vector, and assigning that unit
vector to the `NORMAL` built-in attribute. After setting the *normal map* in the 
*Inspector* to the uniform, you should end up with an animation being lit just like
the previous example.

### Normal Maps with Skeleton Deform

For this example, we are going to use the same sprite and normal map from the first 
example. If you are not familiar with 2D Skeleton Deform, you should refer to 
[2D Skeletons](https://docs.godotengine.org/en/stable/tutorials/animation/2d_skeletons.html)
tutorial from Godot's official documentation before going on, as explaining that
thecnique is out of the scope of this tutorial.

Create a new scene, and this time add a `Polygon2D` node, instead of a `Sprite`. 
Set the texture from the first example to it. Also, add an `Skeleton2D` with a 
`Bone2D` child, and an `AnimationPlayer` to create the animation. Also, as 
always, add a `Light2D` and a `CanvasModulate` node. Your scene should look like 
this:

![Scene tree for skeleton example](skeleton-scene-tree.png)

After that, create an animation like the following one, with the concepts shown in
the tutorial linked above.

{{< video "videos/skeleton-animation.mp4" "720" >}}

You surely noticed that `Polygon2D` doesn't have a *Normal Map* attribute. Then, 
we should do the same we did in [Normal Maps with AnimatedSprite](#normal-maps-with-animatedsprite).
Add a material to the `Polygon2D`, with the same shader of the previous example.
You should see this as a final result:

{{< video "videos/skeleton-with-normal.mp4" "720" >}}

## Normal maps in TileMaps

Once you've added *normal maps* to every sprite in your game, you would probably 
want that your *Tile Maps* react in a similar way to lighting. 

Create a new scene, and this time add a `TileMap`, a `Light2D` and a 
`CanvasModulate` node. Select the `TileMap` in the *Scene Tree*, and in the 
*Inspector* create a new *TileSet*. The *TileSet* editor should open, and you can 
drag and drop there the tileset image. Go ahead and create individual tiles from it.
If you are not familiar with this, refer to [Using tilemaps](https://docs.godotengine.org/en/stable/tutorials/2d/using_tilemaps.html) 
page in Godot's documentation.

Once you created a tile, you should see in the *Inspector* a group called 
*Selected Tile*, and you can drag and drop the *normal map* in the corresponding 
property there. But have in mind that you should set the full *normal map* 
(i.e the *normal map* of the whole tileset) to each single tiles. So you will end up
adding the same *normal map* multiple times, once for each tile. Remember that we 
concluded that the engine uses the same coordinates for the texture and normal map.
This means that if a single tile is using a region of the full tileset, the engine 
will use the same region of the *normal map*. A common mistake here is to set the
*normal map* of the single tile.

Go ahead and continue creating the rest of the tiles, and add the *normal map* to
them. If you did it right, you should see something like this:

{{< video "videos/tilemap-with-normal.mp4" "720" >}}

{{< note >}}
If you are using autotile, you will just need to set the normal map once per 
autotile created. If all your tiles are in sepparate images, then you should also
have normals for each tile sepparated.
{{< /note >}}

## Conclusions

If you commpleted this mini-tutorial, now you should be able to use *normal maps* and 
dynamic lighting in your 2D games for several use cases!
