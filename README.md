CUDA Path Tracer
================

**University of Pennsylvania, CIS 565: GPU Programming and Architecture, Project 3**

* Zhenzhong Tang
  * [LinkedIn](https://www.linkedin.com/in/zhenzhong-anthony-tang-82334a210), [Instagram](https://instagram.com/toytag12), [personal website](https://toytag.net/)
* Tested on: Windows 11 Pro 22H2, AMD EPYC 7V12 64-Core Processor (4 vCPU cores) @ 2.44GHz 28GiB, Tesla T4 16GiB (Azure)

---
## Overview

![](img/stanford-bunny.2023-10-11_16-37-58z.1000samp.png)
Stanford Bunny with Blue-tinted Glossy Material in Cornell Box, rendered in $2000 \times 2000$ resolution with $1000$ samples per pixel and $16$ light bounces.

A path tracer is a rendering algorithm that simulates the behavior of light in a scene. It uses Monte Carlo method to estimate the radiance at each pixel of an image by tracing the path of light through the scene. The algorithm is iterative and parallel in nature, so it runs intuitively and fairly well on CUDA. And it is able to simulate many effects that are difficult with other rendering techniques, such as soft shadows, depth of field, caustics, ambient occlusion, and indirect lighting.

## Features

### Material System

![](img/materials.svg)

Material system is adopted on the above rendering pipeline from [glTF Specification](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#appendix-b-brdf-implementation).

- **Albedo**: The color of the material.
- **Metallic**: The ratio of diffuse and specular reflection. A value of $0$ means the material is dielectric, and a value of $1$ means the material is metal.
- **Roughness**: The roughness of the material. A value of $0$ means the material is perfectly smooth, and a value of $1$ means the material is pure diffuse reflection.
- **IOR**: The index of refraction of the material. A value of $1$ means the material is vacuum, and a value of $1.5$ is a good compromise for most opaque, dielectric materials.
- **Opacity**: The opacity of the material. A value of $0$ means the material is fully transparent, and a value of $1$ means the material is fully opaque.
- **Emittance**: The emittance of the material. A value of $0$ means the material is not emissive, and a value $ > 0$ means the material is emissive, controlling the brightness of the material.


Using the `metallic` and `roughness` parameter, the material can be either dielectric or metal, and its reflection model can be either diffuse or specular. In addition with multiple importance sampling, the path tracer is able to render imperfect specular materials and produce better roughness effect. Also, by controlling the `ior` and `opacity` of dielectrics, the material can produce glass-like refraction with fresnel effect.

|| Diffuse | Imperfect Specular | Pure Specular |
|-:| :----------: | :-----------: | :----------------: |
|**Dielectric**| ![](img/cornell.2023-10-12_00-04-31z.2000samp.png) | ![](img/cornell.2023-10-12_00-11-43z.2000samp.png) | ![](img/cornell.2023-10-12_00-05-52z.2000samp.png) |
|**Metal**| ![](img/cornell.2023-10-12_00-21-21z.2000samp.png) | ![](img/cornell.2023-10-12_00-17-32z.2000samp.png) | ![](img/cornell.2023-10-12_00-23-53z.2000samp.png) |
|**Glass**| ![](img/cornell.2023-10-12_01-07-37z.2000samp.png) | ![](img/cornell.2023-10-12_00-55-54z.2000samp.png) | ![](img/cornell.2023-10-12_00-47-47z.2000samp.png) |

Use the material system, we can mimic many real-world materials. For example, we have the following materials like aluminum, titanium, stainless steel, and different glasses.

| More of Metal | More of Glass |
| :----: | :---: |
| ![](img/cornell-metal.2023-10-12_01-18-06z.2000samp.png) | ![](img/cornell-glass.2023-10-12_01-14-29z.2000samp.png) |

All scenes rendered in $800 \times 800$ resolution with $2000$ spp and $8$ light bounces.

### Anti-Aliasing

Anti-aliasing can be achieved by jittering rays within a pixel. In the following example, the image is rendered in low resolution to exaggerate the effect.

| AA OFF | AA ON |
| :----: | :---: |
| ![](img/aa.2023-10-11_23-07-26z.2000samp.png) | ![](img/aa.2023-10-11_23-08-21z.2000samp.png) |

All scenes rendered in $200 \times 200$ (up-sampled to $800 \times 800$) resolution with $2000$ spp and $8$ light bounces.

### Physically-Based Depth-of-Field

Depth-of-field can be achieved by jittering rays within an aperture. In the following example, the aperture is modeled as a circle with a radius of $0.5$ and the focal length is $10$.

| DoF OFF | DoF ON |
| :-----: | :----: |
| ![](img/dof.2023-10-11_17-23-28z.2000samp.png) | ![](img/dof.2023-10-11_17-22-50z.2000samp.png) |

rendered in $800 \times 800$ resolution with $2000$ spp and $8$ light bounces.

### Procedural Textures

Procedural textures can be achieved by using the `uv` coordinate of the intersection point. Check out the following example.

| Gradient Mario | Checkerboard Mario |
| :-----: | :----: |
| ![](img/wahoo.2023-10-12_01-32-39z.1000samp.png) | ![](img/wahoo.2023-10-12_01-35-52z.1000samp.png) |

All scenes rendered in $800 \times 800$ resolution with $1000$ spp and $8$ light bounces.
