# 3D Fireworks Particle Simulator

> A real-time OpenGL particle system that simulates fireworks exploding into user-defined 3D mesh shapes — guided by physics-based velocity formulas and K-means cluster targeting.

[![C++](https://img.shields.io/badge/C++-17-blue.svg)](https://isocpp.org/)
[![OpenGL](https://img.shields.io/badge/OpenGL-3.3-green.svg)](https://www.opengl.org/)
[![CMake](https://img.shields.io/badge/build-CMake-red.svg)](https://cmake.org/)

## 🎆 Demo

| Bunny | Teapot |
|:---:|:---:|
| <img width="550" alt="bunny" src="https://github.com/user-attachments/assets/3bedde85-50b3-452c-ad24-f1b2e3c92cee" /> | <img width="550" alt="teapot" src="https://github.com/user-attachments/assets/59454369-762f-4484-bd13-7384175d7957" /> |
| **Explosion Teapot** | **Hand** |
| <img width="550" alt="explosion_teapot" src="https://github.com/user-attachments/assets/ac100fd3-d905-42f7-a9de-7f20a17533df" /> | <img width="550" alt="hand" src="https://github.com/user-attachments/assets/bdd31dc6-5dab-4dc4-a457-827b11fc0ce8" /> |
## 📌 Overview

This project implements a real-time 3D fireworks simulator in C++ and OpenGL where the explosion particles don't disperse randomly — they converge into the surface of an arbitrary 3D mesh. Load any `.obj` file, and the firework will reshape itself to match it.

The core technique follows the shape-constrained fireworks system described in **Zhao, H., Fan, R., Wang, C. C. L., Jin, X., & Meng, Y. (2009). *Fireworks controller.* Computer Animation and Virtual Worlds, 20, 185–194. [https://doi.org/10.1002/cav.287](https://doi.org/10.1002/cav.287)**. The paper proposes using physics-based velocity formulas that steer each particle toward a target 3D position while still respecting gravity and air resistance. This implementation realizes that system using OpenGL instanced rendering for efficiency, and K-means clustering to sample target positions evenly from the mesh surface.

## ✨ Key Features

* **Shape-constrained particle explosion:** Each of 5,000 particles is assigned a target K-means cluster center on the loaded `.obj` mesh. Velocity constants `CY` and `CXZ` are precomputed per particle so that it arrives at its cluster position exactly at time `T₀` (4 seconds), then fades out at `T` (5 seconds).
* **Physics-based velocity model:** Particle motion uses the closed-form formulas from Zhao et al. (2009) that account for mass `M`, gravity `g = 9.81`, air friction coefficient `k = 0.5`, and per-particle travel distance `S`. Y-axis and XZ-axis velocities are computed separately since gravity only applies vertically.
* **K-means mesh sampling:** Implemented from scratch in `textures.cpp`, the K-means algorithm clusters the `.obj` vertex positions into `CLUSTER_NO` groups and returns cluster centers as the particle target positions. This ensures even coverage of the shape surface regardless of vertex density.
* **GPU-instanced rendering:** Uses `glDrawArraysInstanced` to render all particles in a single draw call. Three VBOs stream per-particle position/size and color data to the GPU each frame, with particles sorted back-to-front by camera distance for correct alpha blending.
* **Billboard particles with DDS texture:** Each particle is a camera-facing quad textured with a `particle.DDS` sprite, aligned using the camera right and up vectors passed as uniforms to the vertex shader.
* **Interactive camera:** Arrow keys translate the viewpoint; the camera orbits the firework so the 3D shape is visible from all angles.

## 🛠️ Tech Stack

* **Core Language:** C++
* **Graphics API:** OpenGL 3.3 Core Profile
* **Windowing & Input:** GLFW 3.1.2
* **Extension Loading:** GLEW 1.13.0
* **Math:** GLM 0.9.7.1
* **Build System:** CMake
* **Shaders:** GLSL (vertex + fragment)

## 🚀 Getting Started

### Prerequisites

* CMake 2.6+
* OpenGL 3.3-compatible GPU and drivers
* GLFW, GLEW, and GLM (bundled in `external/`)

### Installation

```bash
git clone https://github.com/armanheydari/Simulation-3D-Fireworks-Based-on-Particle-System.git
cd Simulation-3D-Fireworks-Based-on-Particle-System

mkdir build && cd build
cmake ..
make
```

### Running

```bash
./teapot
```

The simulator launches with `teapot.obj` as the target shape. To use a different mesh, replace the `.obj` path in `src/teapot.cpp`:

```cpp
loadOBJ_indexed("../src/meshes/teapot.obj", vertices, uvs, normals, ...);
```

## 💻 How It Works

1. **Load mesh** — an `.obj` file is parsed and its vertices are passed to the K-means algorithm.
2. **Cluster** — K-means iteratively assigns vertices to `CLUSTER_NO` centers and updates them until convergence.
3. **Initialize particles** — 5,000 particles spawn at position `(0, 10, 0)` with life 5s. Each is assigned a cluster target; `CY` and `CXZ` velocity constants are computed from the Zhao et al. formulas using the particle's mass, lifetime, and distance to its cluster.
4. **Update per frame** — `CalculateYVelocity` and `CalculateXZVelocity` advance each particle's position. After `clusterTime` (3s), the particle has arrived at its target; it then continues fading until `life` expires.
5. **Render** — particles are sorted by camera distance, position/color data is streamed to GPU buffers, and all quads are drawn instanced in one call.

## 📁 Project Structure

```
3D-Fireworks-Particle-Simulator/
├── src/
│   ├── teapot.cpp                      # Main: particle system, K-means integration, render loop
│   ├── TransformVertexShader.vertexshader   # Billboard alignment, MVP transform
│   ├── ColorFragmentShader.fragmentshader   # DDS texture sampling
│   └── meshes/                         # .obj target shape files
├── common/
│   ├── shader.cpp / .hpp               # GLSL shader loader
│   ├── controls.cpp / .hpp             # Keyboard camera controls
│   └── objloader.cpp / .hpp            # .obj file parser
├── external/                           # GLFW, GLEW, GLM vendored dependencies
├── CMakeLists.txt                      # Build configuration
├── Methods.pdf                         # Full algorithm and implementation writeup
├── Presentation.pdf                    # Slide deck with physics formulas and results
└── Proposal.pdf                        # Original project proposal and reference summary
```

## 📄 Reference

Zhao, H., Fan, R., Wang, C. C. L., Jin, X., & Meng, Y. (2009). **Fireworks controller.** *Computer Animation and Virtual Worlds*, 20, 185–194. [https://doi.org/10.1002/cav.287](https://doi.org/10.1002/cav.287)
