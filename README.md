# C++ Raytracer Project Documentation for CSCI 711 - Global Illumination
By Audrey Fuller (alf9310@rit.edu)

## Raytracer Assignment #2: Raytracing Framework

The goal of this phase was to implement a non-recursive raytracer.
### The Framework:
![Phase 2 Framework](./phase_2_framework.png?raw=true "Phase 2 Framework")

The World Class holds a C++ vector of Objects, which store:
- A Color (going to be changed to material in phase 3)
- Location and Size parameters (ex: center and radius for a sphere)
- An Intersect function, when given a ray outputs true if there was a collision with the object
- It also populates an "Intersection" object with the point of intersection, normal at the intersection point,
  distance along the ray to the intersection point and color of the object at the intersection point.
- It also would have a transform function to convert all Objects to camera coordinates, but I decided to keep
  everything in world coords for now (sorry Prof!). 

The Camera class is the densest part of the program currently, and stores:
- Camera parameters, such as origin, lookat and up
- Image parameters, such as image aspect ratio, height and width (in pixels)
- Viewport parameters, such as width, height and location (in world coordinates)
- It has a function that generates rays, looping through each pixel and writing the output color to std::cout
  in .ppm format.
- It also has some functions for translating and rotating the camera around the scene (which I used to create the gif below)

The main driver code for the project is Raytracer.cpp, which:
- Populates a World with Objects
- Places and sets up a Camera
- Then renders the scene by calling the camera's generate_rays function on the World

###Example Outputs:
![Spheres](./sphere_image.png?raw=true "Spheres Image")

![Rotating Camera on Spheres](./sphere_loop.png?raw=true "Rotating Camera on Spheres")

For the GIF, I used the Camera's rotate_around_lookat method, which calculates a rotation matrix for y and then updates the 
origin of the camera. This was repeated between 0 and 360 degrees in intervals of 10 degrees, and took aproximately 2 minutes
for the 1,200 x 675px above (though only a few seconds for a 400 x 225 pixel image). I stiched the .ppm images together with 
the Image Magick CLI (Which has the most amazing website ever), which I'm planning on writing a bash script for in the future 
as it was a bit tedius.

###Extra Shapes:
![Cylinders](./cylinder_image.png?raw=true "Cylinder Image")

For the cylinder implementation, I multiplied the Object color by it's z position to show that it wasn't drawing both the base 
and top (which is why the colors look a bit wierd). I based my ray intersect code for it on Inigo Quilez's shadertoy example, 
see here: https://iquilezles.org/articles/intersectors/. There's a small bug with my implementation where the cylinder will apear
warped when close to the boarders of the image, which I'm still trying to figure out the cause of.


## Raytracer Assignment #1: Setting the Scene

For the geometry of this scene, I used Blender's own raytracing engine to render the example shown in the slides.
![Raytracer Blender Scene](./blender_raytracer_output.png?raw=true "Raytracer Blender Scene")

In terms of scene parameters, here were my object transform properties (Z is considered upwards, coordinates are in x,y,z format):

### Sphere 1:
- Size: (1m, 1m, 1m)
- Location (-3m, 0m, 2.5m)
### Sphere 2:
- Size: (1m, 1m, 1m)
- Location (-2m, 3m, 2m)
### Floor:
- Size: (1m, 1m, 1m)
- Location (10m, 50m, 0m)
### Light:
- Location (-3m, -1m, 10m)
### Camera:
- Location (-3m, -9m, 2m)
- Lookat/Rotation (90 degrees, 0, 0)
