# C++ Raytracer Project Documentation for CSCI 711 - Global Illumination
By Audrey Fuller (alf9310@rit.edu)
(Sorry that the design of this site is so bad I don't know how to use github pages yet)

# Raytracer Assignment #1: Setting the Scene

For the geometry of this scene, I used Blender's own raytracing engine to render the example shown in the slides.
![Raytracer Blender Scene](./images/blender_raytracer_output.png?raw=true "Raytracer Blender Scene")

In terms of scene parameters, here were my object transform properties (Z is considered upwards, coordinates are in x,y,z format):
Sphere 1:
- Size: (1m, 1m, 1m)
- Location (-3m, 0m, 2.5m)
Sphere 2:
- Size: (1m, 1m, 1m)
- Location (-2m, 3m, 2m)
Floor:
- Size: (1m, 1m, 1m)
- Location (10m, 50m, 0m)
Light:
- Location (-3m, -1m, 10m)
Camera:
- Location (-3m, -9m, 2m)
- Lookat/Rotation (90 degrees, 0, 0)
