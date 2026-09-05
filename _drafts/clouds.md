---
layout: post
title: "Clouds"
excerpt: ""
date: 2026-09-05
---

Have you ever wondered how volumetric materials are rendered in video games. How do they actually work, especially in early stages of your game related programming journey, chances are you felt perplexed as to how they're made to work. Well that is what we're going to demistify in this post, in addition we're going to see how to optimize them and also make them interactable in real-time without paying for performance -- just like the game Sky: Children of light. This game has played a paramount role for the writing of this post.  

### Ray marching 3D textures
If you're reading this, there is a big chance you have used textures in one of your programs. And there is even a bigger chance that the textures are two dimensional. Meaning to sample color or any kind of information you used uv-coordinates. Now imagine 2D textures stacked together giving it depth, hence we end up with 3D textures. For 3D textures we use uvw-coordinates, with w specifying the depth at which we're sampling. Recent hardwares have built-in ability to process 3D textures, performing trilinear interpolation for smoother results. By now it's clear on how to represent volumetric data, the natural following question should be -- How do we simulate it.  
That's where ray marching comes in to play. Ray marching is an algorithm of various uses, but in general it's used to construct intricate looking shapes, which would be difficult otherwise. It's usually associated with Signed Distance Fields (SDFs), but can also be used with depth data as we will see later.  
If you're familiar with ray tracing, which is shooting rays in order to sample data, you can imagine ray marching the same but stops to sample every amount of specified distance (could be dynamic or static) until it reaches its end. Remember how a 3D texture was defined earlier, if we bring that definition and combine it with our ray marching definition, we can then say -- It's possible to construct a volumetric material by sampling the depth and computing the amount of light that comes in and goes out every step of the way starting from the moment the ray enters the volume container upto the point it exits.  
For simplicity purposes we assume the volume container is a unit cube. And the textures are stacked inside, if we sample for depth information anywhere inside the cube, we should get a smooth and valid number since our hardware assists us in the interpolation process.