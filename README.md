# C++ Raytracer Project Documentation for CSCI 711 - Global Illumination
By Audrey Fuller (alf9310@rit.edu)

## Raytracer Assignment #4: Procedural Shading

The goal of this phase was add the ability to support procedural textures to the raytracer, as well as images :)

![Joe Sphere](./media/joe_geigel_sphere.png?raw=true "Joe Sphere") 

### Texture Class

The largest change to my project structure in this phase was the implementation of the Texture class. 
Instead of an object's Material just storing the base color and illumination parameters, it stores a 
pointer to a texture instead. While every texture child stores UV coordinates, they each use them 
differently when calculating a base color for an intersection in the illumination model. Here are the 
ones I have implemented:
- **Solid Color**: Just a solid color for the entire object instance (does not rely on UV)
- **Checkerboard**: A proceduraly generated checkerboard pattern calculated by if U + V is even or odd.
  Also takes in width parameters for the tiles.
  ![Checkerboard](./media/checkerboard.gif?raw=true "Checkerboard") 
- **Mandelbrot Fractal Texture**: I had to do a bit of research to figure out what this even was before
  trying to implemnet it. In the end, I decided to make my editable parameters for the Mandelbrot set
  equation loop to be the # of itterations to traverse through the set, and two colors (one for when the
  point is in the set, and one for when it isn't.
  ![Mandelbrot](./media/mandelbrot.png?raw=true "Mandelbrot")
- **Perlin Noise Texture**: For this I created a seperate "noise" class that could be used for a variety of
  generation methods (like simplex possibly in the future). I started by doing some reaserch to refresh myself
  on the algorithm, then implementing noise generation in 1D before moving onto 2D & 3D noise (the implementation
  below is with 2D noise mapped to uv).
  ![Noise](./media/noise.png?raw=true "Noise")
- **Image Texture Mapping**: As you might have seen at the top of this section, I finished implementing image
  texture mapping! For this it seemed like the simplist and most lightweight way to read in textures
  would be to include the stb_image & stb_image_read libraries, and then pre-load them into an array of colors
  when defining the material object. Once I debugged my texture display code, I decided to create some more complex
  scenes for fun, like this solar system one:
  ![Solar System](./media/solar_system.gif?raw=true "Solar System")
- **Skybox Cubemap**: While not directly a texture type, the creation of the Solar System Scene above lead to me
  wanting to make the background more interesting by adding stars. So I decided to add a couple absitraction layers
  to my triangle object class, including a plane class comprised of two triangles, then a skybox comprised of six
  planes (so I woudln't have to re-define all 12 triangles every time I want to create a skybox). Currently this
  is about 80% functional, as there seemes to be an issue with each face of my skybox textures lining up, and for
  some reason including a skybox breaks my lighting...
  ![Skybox](./media/skybox.png?raw=true "Skybox")


### Future Additions

I still hope to potentially implement bump mapping, as well as simplex noise generation.


## Raytracer Assignment #3: Basic Shading

The goal of this phase was add an Illumination Model and Shadow testing to the ray tracer.

![Tri Light Phong Model](./media/hq_tri_light_phong.gif?raw=true "Tri Light Phong Model") 

### Supersampling

![No Anti-Aliasing](./media/no_anti_aliasing.png?raw=true "No Anti-Aliasing") ![Random Anti-Aliasing](./media/random_anti_aliasing.png?raw=true "Random Anti-Aliasing")
![Grid Anti-Aliasing](./media/grid_anti_aliasing.png?raw=true "Grid Anti-Aliasing") ![Jittered Grid Anti-Aliasing](./media/jittered_grid_anti_aliasing.png?raw=true "Jittered Grid Anti-Aliasing")

Before I got to implementing an illumination model, I experimented with a few different anti-aliasing methods.
Here they are:
- **None**: Very jagged and uneven edges, still the most efficient as it doesn't require the spawning of any
  additional rays.
- **Random**: Spawing rays through random locations in the pixel, and averaging the results. This one was
  pretty bad and resulted in a lot of wierd colors and rough edges on my shapes (as sometimes the
  random rays would all be in a single area).
- **Grid**: Split the pixel into a "samples" x "samples" grid, and send a ray through the center of each sub-pixel.
  I think this produced the best results for the runtime cost.
- **Jittered Grid**: Same as the grid method, but I added a small random "jitter" offset so it wouldn't always send
  a ray through the center of the sub-pixel. Could be considered better as it added a bit of non-uniformity, but
  the computational cost of generating and adding the offset wasn't worth it IMO.

One algorithm that I was really interested in but didn't quite have time to implement was **Adaptive Supersampling** 
(where you shoot rays through each of the four pixel corners, and if their luminance is different enough you 
subdivide the pixel and try again). Unfortunately my try at an implementation had something wrong with the recursive 
call, and ended up getting stuck in infinate loops :(.

### Illumination Models

![Lambertian](./media/lambertian.png?raw=true "Lambertian") ![Phong](./media/phong.png?raw=true "Phong") ![Phong-Blinn](./media/phong_blinn.png?raw=true "Phong-Blinn") 

Same with the supersampling, I tried to implement a few different illumination models in an "illumination_model" 
class. They are as follows:

- **Lambertian**: Just a simple shadow-ray and diffuse light (light direction dot normal) calculation.
- **Phong**: The best looking model I got working IMO. Added a specular highlight based on the reflection direction.
  Is also the model I used for the GIF at the top of the section.
- **Phong-Blinn**: Using the halfway vector for the specular light is _supposed_ to look more realistic, but the
  lack of a curve makes it look worse than just Phong. I also read that using the halfway vector was supposed to
  improve performance as the reflection doesn't have to be calculated, but in my implementation performance seems
  about the same. There seems to be a strange cut-off for the diffuse component of the light source directly
  above the sphere that I coudn't figure out as well.

I also tried getting the Ashikhmin-Shirley and Cook-Torrence Models running, but ultimately ran out of time. There
was a cool bug I discovered in my half-completed Cook-Torrence code that resulted in this stained-glass effect:

![Wierd Cook-Torrance Bug](./media/wierd_cook_torrance_bug.png?raw=true "Wierd Cook_Torrance Bug")

### Tone Reproduction

Last minor thing I did was saving my outputted Luminance values to a buffer, to then convert to RGB and print to
standard out. This helped me correct really dull results such as the image below. Saving the values to a buffer 
also allowed me to do multi-threading for calculating the pixel values themselves, which sped up the program.

![Twilight Filter](./media/twilight_filter.png?raw=true "Twilight Filter")

## Raytracer Assignment #2: Raytracing Framework

The goal of this phase was to implement a non-recursive raytracer.
### The Framework:
![Phase 2 Framework](./media/phase_2_framework.png?raw=true "Phase 2 Framework")

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

### Example Outputs:
![Spheres](./media/sphere_image.png?raw=true "Spheres Image")

![Rotating Camera on Spheres](./media/sphere_loop.gif?raw=true "Rotating Camera on Spheres")

For the GIF, I used the Camera's rotate_around_lookat method, which calculates a rotation matrix for y and then updates the 
origin of the camera. This was repeated between 0 and 360 degrees in intervals of 10 degrees, and took aproximately 2 minutes
for the 1,200 x 675px above (though only a few seconds for a 400 x 225 pixel image). I stiched the .ppm images together with 
the Image Magick CLI (Which has the most amazing website ever), which I'm planning on writing a bash script for in the future 
as it was a bit tedius.

### Extra Shapes:
![Cylinders](./media/cylinder_image.png?raw=true "Cylinder Image")

For the cylinder implementation, I multiplied the Object color by it's z position to show that it wasn't drawing both the base 
and top (which is why the colors look a bit wierd). I based my ray intersect code for it on Inigo Quilez's shadertoy example, 
see here: https://iquilezles.org/articles/intersectors/. There's a small bug with my implementation where the cylinder will apear
warped when close to the boarders of the image, which I'm still trying to figure out the cause of.


## Raytracer Assignment #1: Setting the Scene

For the geometry of this scene, I used Blender's own raytracing engine to render the example shown in the slides.
![Raytracer Blender Scene](./media/blender_raytracer_output.png?raw=true "Raytracer Blender Scene")

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
