# COMP 371 - 3D Interactive Graphics Application

## Project Title: **[Your Project Name Here]**

**Team Members:**
- Mostafa HASSAN (40228863)
- Mathieu CHOUEIRY (Student ID)
- Sudhansu MOHANTY (Student ID)

## Description

This is a real-time 3D interactive application developed as part of the COMP 371 course. The application implements a scene with:

- Interactive camera controls
- Dynamic lighting (Phong shading)
- Hierarchical animation
- Complex 3D model loading
- Textured surfaces
- Dynamic shadows
- Interactive elements that respond to user input

The application was built using only the following libraries:
- OpenGL
- GLFW
- GLEW
- GLM
- stb_image.h

## Controls

### Camera Controls
- `W`: Move camera forward
- `S`: Move camera backward
- `A`: Move camera left
- `D`: Move camera right
          - `Q`: 
          - `E`: 
- `Shift`: Hold to move faster
- Mouse Movement: Look around

### Scene Interaction
- `Left Mouse Button`: Interact with objects (e.g. select, trigger animation)
- `1` / `2` / `3`: Switch between different camera modes
                      - `R`: Reset
- `Esc`: Quit the application

## Features

### Assignment 1 Requirements
- Interactive camera with mouse and keyboard
- Movement relative to camera direction
- Multiple textured surfaces
- One of:
  - Dynamic moving lights
  - Hierarchical animation

### Assignment 2 Enhancements
- Dynamic lights using Phong shading
- Hierarchical animation (e.g., robot arm, orrery)
- Loaded complex 3D models from `.obj` files
- Dynamic shadows from moving lights
- Scene objects that respond to user input
- Creative enhancements (e.g., particle effects, UI overlay, sound effects)

## Building the Project

### Requirements
- C++17 or higher
- GLFW
- GLEW
- OpenGL
- GLM
- stb_image.h

### Compilation (example using g++)
```bash
g++ main.cpp -o Project -lglfw -lGLEW -lGL -ldl
./Project
