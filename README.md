# C++ Raytracer Project Documentation for CSCI 711 - Global Illumination
By Audrey Fuller (alf9310@rit.edu)

## Raytracer Advanced Assignment: KD-Tree Spacial Data Structure

![Bisexual Man](./media/bisexual_man.png?raw=true "Bisexual Man") 

![Glass Man](./media/glass_man.png?raw=true "Glass Man") 

The goal of this phase was speed up the ray tracer by introducing the use of a spatial data structure, 
namely a k-d tree.

### Reading 3D Objects into the Scene

Before adding an acceleration structure to render complex scenes quicker, I wanted my raytracer 
to first be able to actually support complex scenes! So instead of hand writing thousands of polygon 
coordinates myself, I used the open plycpp library (2021 Romain Brégier) to read in Stanford 3D scanned 
models. Then with some intermediate code to convert the plycpp output to the triangle objects in my 
raytracer, I was able to render a compressed (7,000 polygon) Stanford bunny in a whopping hour and a 
half. 

![KD Tree Bunny](./media/kd-tree_bunny.png?raw=true "KD Tree Bunny") 

While normally I would have started on the KD-tree by now, I was beginning to get a bit dissadisfied 
with the .ply file format. As the Polygon File Format is most commonly used for 3D scans of objects, 
most of the interesting open-source 3D models made by artists online were instead most commonlt found 
in the .obj file format instead. SO instead of going through the hassle of importing them into Blender 
to then convert to .ply files, I ended up adding support for .obj files as well for my raytracer. For the 
main parsing code, I really liked the 'tiny obj loader' library (2012 Syoyo Fujita) for it's fast file reading. 
Additionally, all of the 3D models shown in this section besides the compressed KD-Tree bunny are from the 
McGuire Computer Graphics Archive ([Link Here](https://casual-effects.com/data/index.html)).

![Big KD Tree Bunny](./media/bunny_144k.png?raw=true "Big KD Tree Bunny") 

### Building the KD-Tree

Now for actually rendering these massive .obj files! For my KD Tree archetecture, I have three main classes:
- **Axis-Aligned Bounding Box**(aabb): Stores two 3D coordinates for the smallest and largest corner values.
- **KD-Node**: Makes up the contents of the KD-Tree, and stores an aabb for it's location in the scene.
Can either be an internal node that holds pointers to front and rear KD-Node children, or a leaf node
which stores a list of pointers to the objects that fall within the confines of the node's bouding box.
- **KD-Tree**: The main spatial partitioning data structure used to speed up ray-triangle intersection calculations,
similar to a binary search tree but over 3 dimentions. Stores a root KD-Node, a list of objects in the tree, and a
list of aabbs for said objects. Also keeps track of current and maximum set depth for building.

Here's the algorithm that my recursive KD-tree building uses:
Given a list of all objects, AABB of all objects & a root
1. If terminal, return a new KDNode (leaf) with the objects
2. Find partition plane (Spacial Medium Method or Surface Area Heuristic)
3. Split the the AABB into two halves (front and rear)
4. Partition the objects into two halves
5. Create child nodes recursively
6. Return a new KDNode (internal) with the child nodes

I initially coded my tree to use the Spacial Medium Method for finding which plane to split along (a simple 
round-robin style choice). However, this lead to some really nonoptimal splits and deep trees that took a 
considerable about of time to traverse. So instead I swapped to the more robust Surface Area Heuristic algorithm, 
which chooses a split axis adaptively based on the scene geometry to minimize the area of triangle-dense nodes. This 
means that rays are less likely to have to traverse through computationally expensive parts of the scene they may not 
intersect with. 

Here's the algorithm:
1. For each axis (x, y, z), gather potential split positions from object AABB min/max bounds.
2. For each position:
3.   Partition objects into front and rear.
4.   Calculate front/rear AABBs and their surface areas.
5.   Evaluate cost using C = 1 + SA(front)/SA(total) * Nf + SA(rear)/SA(total) * Nr.
6.   Track the axis and position with the lowest cost.

While taking a bit longer to build the KD tree, this method lead to some fantastic speed-ups during traversal!

![Reflective Bunny](./media/reflective_bunny.png?raw=true "Reflective Bunny") 

### KD-Tree Traversal

For traversing the KD-tree when checking for ray-polygon intersections, I use a TA-B algorithm for additional 
speed. While the structure is similar to a TA-A algorithm, it uses coordinates rather than distances to determine 
whether or not to traverse the far node. 

Heres the (final) algorithm used:
1. Base Case – Null Node or if the ray doesn't intersect the node's bounding box
2. Leaf Node Check: Iterate through the node's objects to find the closest intersection, exit if hit.
3. Internal Node – Compute: The splitting axis and position, ray's direction and origin along the split axis, and distance to the split plane.
4. Handle Parallel Rays (Z cases): If the ray is nearly parallel to the split plane only traverse one child
5. Determine Near vs Far Children based on ray direction
6. Decide Traversal Case (P/N cases):
7.   Case 2 (P2/N2): Split is behind entry → traverse far child only.
8.   Case 3 (P3/N3): Split is beyond exit → traverse near child only.
9.   Case 4 (P4/N4): Split is between entry and exit → Traverse near child first, if a hit is closer the split, return early. Otherwise also check far_child.

![Teapot](./media/teapot.png?raw=true "Teapot") 

### Runtime Speedup

While the KD-tree worked fantastic for complex scenes (thousands of objects), due to the Whitted Scene only having four objects, ultimately the creation lead to 
more overhead than traversal time saved. Here's some tables showing both KD-tree building and render time performance for various depths and scene complexities.

![KD-tree Times](./media/kd-tree_times.png?raw=true "KD-Tree Times") 

## Raytracer Assignment #7: Tone Reproduction

The goal of this phase was to apply a decent tone reproduction operator to compress simulated radiances to display radiances. For this I used two methods, 
Ward’s tone reproduction operator (Perceptual) and Reinhard’s tone reproduction operator (Photographic). Here are the results:

Light Intensity Values: 1, 2, 4

### Ward Tone Reproduction 
Simple heuristic based on perceptual tests.

![Ward 1](./media/ward_1.png?raw=true "Ward 1") 
![Ward 2](./media/ward_2.png?raw=true "Ward 2") 
![Ward 4](./media/ward_4.png?raw=true "Ward 4") 

### Reinhard's Tone Reproduction
Simple heuristic based on photographic systems to model a photographic-like response.
![Reinhard 1](./media/reinhard_1.png?raw=true "Reinhard 1") 
![Reinhard 2](./media/reinhard_2.png?raw=true "Reinhard 2") 
![Reinhard 4](./media/reinhard_4.png?raw=true "Reinhard 4") 

## Raytracer Assignment #6: Transmission & Refraction

The goal of this phase was to add support for transparent materials by spawning transmissive rays when hit. 

![Refraction](./media/refraction.png?raw=true "Refraction") 

(One transparent sphere where kt = 0.8 & IOR = 1.03, and one opaque & reflective where kt = 0.0)

### Changes to Material & Illumination Model Classes

Same as the last phase, I added aditional parameters to my Material class. This includes the transmission 
coefficient, which determines the transparency of the material and it's shadow. I also added an additional 
refraction index perameter, which controls the angle of the refraction ray when passing through mediums 
(example IORs 1: air, 1.5: glass, 2.4: diamond). 

The changes to the illumination model are also similar to what was done for reflection, as when a ray hits a 
non-opaque object, it spawns a refraction ray, using Snell's Law to calculate the angle. The first step of this 
was determining if we're entering or exiting the medium using the dot product of the incoming ray and the normal. 
If this is <0, that means we're exiting the material, so the IOR is inverted and the normal is flipped. The other 
special case of total internal reflection is handled by checking if the angle of a ray moving from a high refractive index a lower one is steep. If the sin^2(theta_t) value is suitably large ( >= 1), the light is reflected on the edge 
of the sphere instead. 

Getting transparent shadows to work was a bit trickier, as it involved changing how I spawn in shadow rays. While I 
initially had the idea of multiplying the shadow ray by the transmission property of the material it intersected 
with, this didn't acount for the cases where there may be multiple transparent objects between the shadow and the 
light source. So instead, I spawned shadow rays in a loop, increasing the attenuation of the light by the 
transmission variable of each object it passes through (For example, if it passes through two objects with 50% transparency on the way to the light source, the shadow will be completely solid. While this isn't 100% accurate to 
how real transpatency work (as I should be scaling the shadow by the depth of the material), it seems like a close 
enough approximation. 

### Completed Whitted Scene

Now that all of the components of the original 1980's Whitted Raytracer are completed, here is the closest 
approximation I managed to rended to the origional scene using my engine. 

![Whitted Refraction](./media/whitted_refraction.png?raw=true "Whitted Refraction") 

Original Whitted Scene:
![Original Whitted](./media/original_whitted.png?raw=true "Original Whitted") 

Reference Provided in Class:
![Class Whitted](./media/class_whitted.png?raw=true "Class Refraction") 

Even with the core of my rendering engine complete, there still seems to be a couple element's I'm not able to 
replicate yet. Here they are:
• An extra layer of refraction/rim lighting on the outside of the mostly transparent sphere. This could possibly 
be due to the inclusion of a smaller "air sphere" inside of it, leading to additional refraction rays being spawned. 
• A 'fuzzy' effect for the reflection around the edges of the sphere.  
• The lack of a getting rid of the 'double highlight' on the scene's refractive sphere. 

Depending on how much time I have remaining, I'd like to fix up these issues to get the result looking as close as 
I can!

![Whitted Refraction GIF](./media/whitted_refraction.gif?raw=true "Whitted Refraction GIF") 


## Raytracer Assignment #5: Reflection

The goal of this phase was to add support for reflective materials, by recursively spawning in reflective rays.

![Reflective Random](./media/reflective_random.png?raw=true "Reflective Random") 

### Changes to Material & Illumination Model Classes

The main changes implemented were adding reflection & transmission coefficient variabled to the Materials class 
to determine how much of the reflected ray to apply to the object's color output. Then in the illumination model 
class, I did a bit of refactoring to get rid of some of the nested if-statements & loops for readability, and 
recursively shot out rays in the reflected direction until the maximum depth was reached (being 5 for the below 
image). While this does support multiple reflective rays, it doesn't yet Implement Kajia’s method for stochastic
sampling yet.

![Whitted Reflective](./media/whitted_reflective.gif?raw=true "Whitted Reflective") 


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
