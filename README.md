# Volumetric Screen-Space Fog & Lighting

A real-time volumetric fog and lighting implementation in a custom game engine using OpenGL and DirectX 11.

This project extends the engine's rendering pipeline with **depth-based exponential fog, screen-space ray marching, shadow-map sampling, and animated 3D noise** to simulate atmospheric scattering, light shafts (god rays), and moving dust particles.

The primary goal was to explore how volumetric lighting effects can be approximated in real time using screen-space techniques, while integrating the effect into the engine's existing post-processing pipeline.

> **Repository note:** This repository currently serves as a
> documentation and project-overview landing page. The README describes
> the engine architecture and design foundations; it does not imply that
> the full source code, benchmarks, or test suite are present in this
> repository.

## Overview

Traditional distance-based fog blends scene colors with a fog color depending on the distance between the camera and visible geometry. While effective at creating atmospheric depth, it does not account for how light interacts with the environment.

This implementation builds on depth-based fog by ray marching between the camera and visible scene geometry, sampling shadow maps along the ray to estimate how much of the ray is occluded from the light source.

The resulting shadow information influences the fog color, producing volumetric lighting effects such as god rays and light shafts.

Additional animated 3D noise modulates fog density to simulate moving dust and atmospheric movement.

## Features

- **GPU Depth Map Generation:** Exposes the camera's depth buffer as a GPU texture for post-processing effects, without requiring an additional geometry rendering pass.
- **Exponential Depth-Based Fog:** Uses linearized scene depth to calculate fog contribution and blend the fog with the original scene color.
- **Screen-Space Ray Marching:** Reconstructs world-space positions from screen coordinates and depth, then marches along the camera ray.
- **Shadow-Based Volumetric Lighting:** Samples light-space shadow maps along the ray to estimate the percentage of the ray occluded from the light source.
- **Ray Jittering:** Uses randomized ray offsets to reduce visible banding caused by a limited number of ray-marching samples.
- **Animated 3D Density Noise:** Uses scrolling 3D noise to modulate fog density, creating the appearance of moving dust and wind.
- **Post-Processing Integration:** Combines the volumetric fog contribution with the original scene color in the existing post-processing pipeline.

## Rendering Pipeline

<img width="1148" height="3980" alt="VolumetricPass" src="https://github.com/user-attachments/assets/a1fb823a-86d0-47d8-9d4f-0f3cfa183f18" />

The effect is implemented as a screen-space post-processing pass.

1. The scene is rendered normally, with its depth information stored in a GPU depth texture.
2. The post-processing shader samples the depth texture for each screen pixel.
3. The pixel's screen-space coordinates and depth are used to reconstruct its world-space position.
4. A ray is defined between the camera position and the reconstructed world-space position.
5. Multiple samples are taken along the ray.
6. Each sample is transformed into light space and tested against the shadow map.
7. The shadow samples are averaged to estimate the percentage of the ray occluded from the light.
8. The resulting volumetric lighting contribution is combined with exponential depth-based fog.
9. Animated 3D noise modulates the fog density.
10. The final fog contribution is blended with the original scene color.

## Implementation Details

<img width="1371" height="1562" alt="EngineLevelChanges" src="https://github.com/user-attachments/assets/b371bd91-ae94-4dd6-89de-da6a75aaaead" />

### 1. Camera Depth Map Generation

The first challenge was making the scene's camera depth information available to post-processing shaders.

The engine originally exposed depth textures primarily for shadow mapping, where depth is generated from the perspective of individual lights rather than the main camera.

An initial solution involved rendering the scene's depth information in an additional pass and storing it in a texture. However, this introduced significant rendering overhead.

Instead, I modified the engine's existing OpenGL texture handling to retain the GPU depth texture created as a depth attachment. This allowed the post-processing shader to access the camera's depth information directly, eliminating the additional geometry pass.

### 2. Exponential Depth-Based Fog

With depth available in the post-processing shader, fog could be calculated using the distance of each visible pixel from the camera.

The non-linear depth buffer values are converted into normalized linear depth using the camera's near and far clipping planes.

For the implementation, the camera uses:

- Near plane: `0.05`
- Far plane: `500.0`

The resulting linear depth is used to calculate the fog contribution exponentially based on fog density.

This produces a smooth transition between the original scene color and the fog color, with distant objects becoming increasingly obscured.

### 3. Screen-Space to World-Space Reconstruction

To perform volumetric lighting calculations, each screen-space pixel needs to be reconstructed into a 3D world-space position.

This is achieved using the inverse projection and inverse view matrices.

The pixel's UV coordinates and sampled depth are transformed from screen/clip space into homogeneous world space. After perspective division, the resulting position represents the visible surface point in the scene.

A ray can then be constructed from the camera position to this reconstructed world-space position.

### 4. Shadow Map Sampling and Ray Marching

Once the camera ray is established, multiple sample points are distributed along its length.

Each sample point is transformed into the light's coordinate space and compared against the corresponding shadow map.

This determines whether the sample is illuminated or occluded from the light source.

By averaging the results across all samples, the implementation estimates the percentage of the camera ray that lies in shadow.

This shadow percentage influences the fog color, allowing illuminated and shadowed portions of the ray to contribute differently to the final volumetric effect.

### 5. Ray Jittering and Banding Reduction

A limited number of ray-marching samples can introduce visible banding artifacts, particularly when the sample positions are regularly spaced.

To mitigate this, a randomized offset is applied to the ray-marching positions.

This distributes sample locations rather than keeping them aligned across neighboring pixels. The resulting banding becomes significantly less noticeable, at the cost of introducing a small amount of spatial noise.

The noise offset also provides a potential foundation for future temporal accumulation techniques.

### 6. Animated 3D Density Noise

To make the fog feel less uniform and static, an additional 3D noise texture is sampled to modulate fog density.

The noise coordinates are animated over time, creating the appearance of dust or atmospheric particles moving through the environment.

This produces subtle wind-like movement within the fog and adds variation to the volumetric lighting.

## Architecture
The implementation can be divided into four major stages:

- **Depth Generation:** Produces and exposes the camera depth texture.
- **World Reconstruction:** Converts screen-space pixels into world-space positions and constructs camera rays.
- **Volumetric Integration:** Marches along the rays and evaluates shadow visibility.
- **Fog Composition:** Combines shadow-based lighting, depth-dependent fog, and animated density noise into the final image.

## Technical Challenges & Learnings

This project involved several important rendering and graphics programming concepts:

- Understanding how the engine creates and manages GPU textures and depth attachments.
- Integrating GPU depth information into an existing post-processing pipeline.
- Avoiding unnecessary geometry passes to reduce rendering overhead.
- Reconstructing world-space positions from screen-space coordinates using inverse camera matrices.
- Transforming world-space sample positions into light space for shadow-map lookups.
- Implementing screen-space ray marching and estimating volumetric shadow contribution.
- Balancing ray-marching sample count against rendering performance.
- Reducing sampling artifacts using randomized ray offsets.
- Using animated 3D noise to simulate non-uniform, moving atmospheric density.

## Demo

https://github.com/user-attachments/assets/a6f95b9f-13aa-47af-8edb-9a472aa10b2d

## Technologies

- C++
- OpenGL
- DirectX 11
- GPU Shader Programming
- GLSL/HLSL/CG-style shaders
- Shadow Mapping
- Post-Processing
- Screen-Space Rendering
- Ray Marching
- 3D Noise Textures
