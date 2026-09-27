---
layout: post
title: "Clouds"
excerpt: ""
date: 2026-09-05
---

Have you ever wondered how volumetrics are rendered in video games. How do they actually work, especially in early stages of your game related programming journey, chances are you felt perplexed as to how they're made to work. Well that is what we're going to demistify in this post, in addition we're going to see how to optimize them and also make them interactable in real-time without paying for performance -- just like the game Sky: Children of the Light. This game has played a paramount role for the writing of this post.  

### Ray marching 3D textures
If you're reading this, there is a big chance you have used textures in one of your programs. And there is even a bigger chance that the textures are two dimensional. Meaning to sample color or any kind of information you used uv-coordinates. Now imagine 2D textures stacked together giving it depth, hence we end up with a 3D texture. For 3D textures we use uvw-coordinates, with w specifying the depth at which we're sampling. Recent hardwares have built-in ability to process 3D textures, performing trilinear interpolation for smoother results. By now it's clear on how to represent volumetric data, the natural following question should be -- How do we simulate it.  
That's where ray marching comes in to play. Ray marching is an algorithm of various uses, but in general it's used to construct intricate looking shapes, which would be difficult otherwise. It's usually associated with Signed Distance Fields (SDFs), but can also be used with depth data as we will see later.  
If you're familiar with ray tracing, which is shooting rays in order to sample data, you can imagine ray marching the same but stops to sample every amount of specified distance (could be dynamic or static) until it reaches its end. Remember how a 3D texture was defined earlier, if we bring that definition and combine it with our ray marching definition, we can then say -- It's possible to render a volumetric data by sampling the depth and computing the amount of light that comes in and goes out every step of the way starting from the moment the ray enters the volume container upto the point it exits.  
For simplicity purposes we assume the volume container is a unit cube. And the textures are stacked inside, if we sample for depth information anywhere inside the cube, we should get a smooth and valid number since our hardware assists us in the interpolation process.  

### How to create a 3D texture
Model a cloud the size less than a unit cube inside the modelling software of your choice. Convert that mesh into a volume. Set the camera to orthographic mode and animate the start and end clip of the camera to render parts of the volumetric at a time. Make sure the first and last frames don't include any part of the volume. Set the rendered frames' resolutions to 256x256. After rendering the animation you'll end up with 64 frames containing parts of the volumetric cloud, except the first and last. Note that the the first and last couple of frames can be blank, but the first and last are mandatory if you want a cloud which doesn't look clipped.  
The next step is to put the frames into one single image as a spritesheet, it's also called an atlas texture. I did that using Gimp, which is a free image editing software, but again you can use any software to your liking. We want the atlas texture to be 2048x2048, the rendered images are 256x256, which results in a texture of 8x8 grid of the rendered frames. You can place the frames by hand, but instead of doing that I used a simple custom plugin that does that for me.  
Now this isn't a 3d texture yet, this is still a 2d texture which contains other images inside it. I'll be using specific terms from the Godot engine, but the concept applies in general. To convert it into a 3d texture, you can change the Texture2D into Texture3D in the import settings. You can also do it manually by writing a script that separates out each image from the atlas texture and put it inside an array of images, we then instantiate a 3d texture object by passing the array as one of the arguments along with other data such as volume width, height, and depth. 
![cloud texture]({{ '/assets/images/clouds/cloud_atlas_512.png'}}){: width="40%" .image-center }

### Volume rendering
We have seen how volumetric data is stored and covered the alogrithm we're going to use in order to display the volume data on screen. Rendering volumetric medium means we're going to deal with three lighting phenomenas, namely transmission, absorption, and shadowing. Our algorithm should simulate how much of the light that is passing through the participating medium is making it to the other side, it has to simulate how much of those light rays didn't make it out resulting in their absorption, and finally how much of those were occluded by the other parts of the volumetric medium. This implementation was taken from the book The Godot Shaders Bible, which they derived from a developer by the name DMville which he implemented on Unity for folks who prefer the engine.  
Since a lot is covered in the book, instead of repeating the same information I'll specifically explain how the algorithm works.  
```{python}
	for (int i = 0; i < NUM_STEPS; i++)
	{
		ray_origin += (ray_direction * STEP_SIZE);
		vec3 sampled_position = ray_origin + offset;
		float sample_density = texture(base_tex, sampled_position).r;
		density += sample_density;

		vec3 light_ray_origin = sampled_position;
		light_accumulation = 0.0;
		for (int j = 0; j < NUM_LIGHT_STEPS; j++)
		{
			light_ray_origin += (light_dir * LIGHT_STEP_SIZE);
			float light_density = texture(base_tex, light_ray_origin).r;
			light_accumulation += light_density;
		}

		float light_transmission = exp(-light_accumulation);
		float shadow = darkness * light_transmission * (1.0 - darkness);
		final_light += density * transmittance * shadow;
		transmittance *= exp(-density * light_absorb);
	}

	transmission = exp(-density);
```
A ray is emitted from the camera stepping one STEP_SIZE at a time, sampling the density value. At each step it shoots an additional ray, this time in the direction of the light source with it's origin being the sampled_position. The additional ray is for computing how much light that part of the volume receives. If that ray passes through a dense part of the volume, the light_transmission is lower. Having a strong light_transmission affects the area of volume in shadow, resulting in lit volumetric surface. This process continues until the main ray, the ray which was emmited from the camera takes its final step. At every iteration of this process, as the ray goes through the volume, the transmittance gradually decreases by the specified light_absorbtion value. Finally the overall transmission value is computed, it is inversly proportional to the density that was accumulated.