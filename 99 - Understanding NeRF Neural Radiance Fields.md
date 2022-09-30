![](https://miro.medium.com/max/700/1*WtKQ4pgG1RZQYshSqTvw0w.png)

## Understanding the concepts behind NeRF and Neural Rendering

## Introduction

In this article, we will start with an explanation of what is Computer Graphics (a very short introduction, with a brief on **volume rendering** and **view synthesis).** We will then have an overview of what is Neural Radiance Fields and how it is used in view synthesis.

I will also give the model used by NeRF in a gist code that can be used to get the code running. But remember that it is not the entire code so if you copy it and try to run it in your code the probability of your code working is negligible. In case you are looking for the source code for NeRF then it can be checked here in this [GitHub](https://github.com/bmild/nerf) link.

## Computer Graphics Introduction

This section is solely for people who are new to computer graphics and may not be familiar with the terminology and definitions of various techniques used in this area. So if you are familiar with this topic feel free to skip through this section.

Before we can jump on to Neural Radiance Fields it is good to be familiar with different areas of computer graphics. Computer Graphics (as per Wikipedia) is a branch of computer science that deals with generating images with the aid of computers. Now the question that might come is why focusing on only images and not videos? Well, the answer is simple a video is nothing but a time-series of images (FPS: Frame Per Second) in a sequence. So when you are running a game (let’s say Arkham Asylum) then, at that time your GPU computes scenes as an image. What you see next is your character (let’s say Batman) interacting with an environment (Jumping from Buildings, driving Batmobile, punching joker ) in terms of frames (a collection of images) that is being rendered per second for each of those scenes. So for simplicity, we can say that videos are nothing but several images in time duration (imagine it like a flipbook or flick book).

Now computer graphics is a very vast area in itself comprising of multiple topics like UI design, rendering, ray tracing, vector graphics, geometry processing, 3D modeling, image processing, shading, Material and texture, and so on. This is not a tutorial for computer graphics so please feel free to research the different topics. Every single topic mentioned above is an area of specialization in itself. I will rather be focusing on two things that too in the brief first one is rendering and another is view synthesis. So let’s get started.

## Rendering

Rendering or image synthesis is a process of generating an image from a 2D or 3D model using a computer program. The resulting image is called render. Rendering is usually the last step in the graphics pipeline which gives models and animations their final appearance. Any rendering application gets an _input file_ called a _scene file__._ This scene file contains multiple information like for example:

-   Model (3D or 2D model itself)
-   Texture
-   Shading
-   Shadows
-   Reflection
-   Lighting
-   Viewpoint

The information stated above is considered as a _feature_ for rendering. It means we can say that each _scene file contains multiple features_ that need to be understood and processed by the rendering algorithm or application to generate a processed image.

NOTE: It means all these steps (Model designing, texture, shading, lighting, and viewpoint) has been executed before the rendering stage, and now rendering gets a single input file which needs to be processed with all this information and generate an output image.

NOTE: Also, on a side note many people think that in video games any character and background scene gets rendered as a single file. This is incorrect because if we talk about video games remember your character (Batman) is not a part of a single model file that comprises of character and the environment. Rather the scene or environment is loaded proactively by your GPU (remember loading times that is when it is happening). Batman and other interacting elements (like Bat Mobile) will be a part of separate individual files that will be rendered actively as the game progresses by your GPU. This is important to be cleared out because as we will see that our neural rendering technique (NeRF) is focusing on the environment and not the character.

The rendering algorithm or technique which tries to solve the problem of image generation based on all given features is mostly trying to solve for **_rendering equation_** in the most optimized time. Now I will not explain what this equation is and how to solve it please follow [**this link**](http://graphics.stanford.edu/courses/cs348b-11/lectures/renderingequation/renderingequation.pdf) to have a look at this equation. At a high-level rendering equation essentially computes for a radiance that is illumination (reflection, refraction, and emittance of light) on an object from a source to an observer in a given space. The reason I brought this up is that NeRF is computing for volume rendering. Also, the algorithms for solving the rendering equation can be loosely classified under three categories:

-   Rasterization: Including [scanline rendering](https://en.wikipedia.org/wiki/Scanline_rendering), This algorithm geometrically projects objects in the scene to an image plane, without advanced optical effects
-   R[ay Casting](https://en.wikipedia.org/wiki/Ray_casting): It considers the scene as observed from a specific point of view, calculating the observed image based only on geometry and very basic optical laws of reflection intensity, and perhaps using [Monte Carlo](https://en.wikipedia.org/wiki/Monte_Carlo_method) techniques to reduce artifacts
-   [Ray Tracing](https://en.wikipedia.org/wiki/Ray_tracing_(graphics)): It is similar to ray casting, but employs more advanced optical simulation, and usually uses Monte Carlo techniques to obtain more realistic results at a speed that is often orders of magnitude faster (Nvidia uses this technique in their GPUs)

## Volume Rendering

We already saw what is rendering in brief. Now let us see a brief about what is volume rendering. Volume rendering (as per Wikipedia) is a set of a technique used to display a 2D projection of a 3D discretely sampled data set. Now to render such a 2D projection (output) of a 3D dataset we first need to define the camera position in space relative to the volume then we need to define the RGBα (Red, Green. Blue, Alpha → it stands for opacity channel) for every voxel. The camera position can be either of the following

-   Fixed: The developers can set the properties of the camera, such as its position, orientation or [field of view](https://en.wikipedia.org/wiki/Field_of_view), during the game creation.
-   Tracking: A tracking camera follows the characters from behind. The player does not control the camera in any way — they cannot for example rotate it or move it to a different position.
-   Interactive: While the camera is still tracking the character, some of its parameters, such as its orientation or distance to the character, can be changed. The camera is often controlled by a mouse or analog stick.

NOTE: A Voxel is a value in a regular grid in 3D space (for simplicity consider a single cube element or smallest cube from a Rubik’s cube or you can also say a voxel is similar to what a pixel is in 2D space).

The primary objective in volume rendering is to get a transfer function which defines RGBα for every value for every possible voxel value in a given space.

![](https://miro.medium.com/max/700/1*N95a4cYKTcWgjfF3V0OxfQ.jpeg)

Types of presentations of CT scans, with two examples of volume rendering. (Image Courtesy: Wikipedia)

## View Synthesis

The objective in front of us is to create a 3D view from a 2D scale so the question is how to achieve that. One thing we can do is probably constructed using linear perspectives but then again it does not help in ray tracing and highlighting the features of the object from different camera angles and restrict the view to one or two or three-point perspective at most with a fixed camera angle. See an example below on what perspective means:

![](https://miro.medium.com/max/700/1*bXg9Sto4lgjDUmrmfcGBKA.jpeg)

Staircase in two-point perspective (Courtesy: Wikipedia)

Another thing to do is maybe click photos of an object from multiple camera angles and superimpose the images to have a look at the same object from different known camera angles and positions in a given hemispherical plan of the object itself. This is one of the common techniques of 3D reconstruction from multiple images. For this type of technique, we are trying to predict the third missing axis (the first two being length and breadth) which is the depth. We try to predict a function for depth determination at various points in the plane against the object itself. This is where the Neural Radiance Fields come into consideration.

![](https://miro.medium.com/max/541/1*QdtBGxaXHl9SPBSXX2O8Tg.png)

Generating and reconstructing 3D shapes from single or multi-view depth maps or silhouette (Courtesy: Wikipedia)

## Neural Radiance Fields

NeRF or better known as Neural Radiance Fields is a state-of-the-art method that generates novel views of complex scenes by optimizing an underlying continuous volumetric scene function using a sparse set of input views. The input can be provided as a blender model or a static set of images.

The input is provided as a continuous 5D function that outputs the radiance emitted in each direction (θ; Φ) at each point (x; y; z) in space, and a density at each point which acts like a differential opacity controlling how much radiance is accumulated by a ray passing through (x; y; z).

![](https://miro.medium.com/max/700/1*xw-iGfxSdTTXQLmAJj-8Eg.png)

The figure above represents the steps that optimizes a continuous 5D (x; y; z; θ; Φ) neural radiance field representation (volume density and view-dependent color at any continuous location) of a scene from a set of input images. Here in this example of drum set, approximately 100 images were given as input with various x, y, z, θ, and Φ values for each of them.

A continuous scene can be described as a 5D vector-valued function whose input is a 3D location x = (x; y; z) and 2D viewing direction (θ; Φ), and whose output is an emitted color c = (r; g; b) and volume density (α). To generate a Neural Radiance Field from a particular viewpoint following steps were done:

1.  March camera rays through the scene to generate a sampled set of 3D points
2.  Use those points and their corresponding 2D viewing directions as input to the neural network to produce an output set of colors and densities
3.  use classical volume rendering techniques to accumulate those colors and densities into a 2D image

![](https://miro.medium.com/max/700/1*uE6cqVbIdTGZMMYfMR1Rvw.png)

(a) Sampling 5D coordinates (location and viewing direction) along camera rays; (b) feeding those locations into an MLP to produce a color and volume density; c)using volume rendering techniques to composite these values into an image; (d) optimize the scene representation by minimizing the residual between synthesized and ground truth observed images

NOTE: The Optimization here is happening for a deep fully connected multi-layer perceptron without using any convolutional layers. Also, gradient descent is used to optimize this model by minimizing the error between each observed image and the corresponding views rendered from the representation.

The model (MLP) used to process 5D input into RGBα output can be seen in the gist below:

Once the **RGBα** is obtained we optimize the loss function and reduce the error. So far we have used the model to create an output of color and volume density. We now render the color of any ray passing through the scene using principles from **classical volume rendering**. The function for volume rendering can be seen from [**gist here**](https://gist.github.com/varun-bhaseen/f9489c344d541fa74824cc138e720044).

## Performance

While NeRF works well on images of static subjects captured under controlled settings, it is incapable of modeling many ubiquitous, real-world phenomena in uncontrolled images, such as variable illumination or transient occluders.

To adapt NeRF to variable lighting and photometric postprocessing, Some additional changes need to be done on NeRF which includes Latent Appearance Modeling and Transient Objects which are described in detail by authors in the paper [NeRF-W](https://arxiv.org/abs/2008.02268)

## Conclusion

In this article, I tried to give a brief of what computer graphics is and what is the role and contribution of NeRF. We can summarize by saying that NeRF is doing a reconstruction of scenes by using multiple images as input for a scene. the input is fed to an MLP neural network with a 5D (x; y; z; θ; Φ) input and gives an output of form RGBα. This output of RGBα is taken as input by any volume rendering algorithm to generate a final view of the scene. The entire sequence can be termed as neural rendering and view synthesis.

Please feel free to check out the page of the [NeRF project](https://www.matthewtancik.com/nerf) you can also download the dataset from there as well. Check out the [GitHub link](https://github.com/bmild/nerf) for cloning and replicating the results.

There is an additional paper titled [NeRF in the wild](https://arxiv.org/abs/2008.02268) which uses the NeRF model as a base and further enhances for real-world applications for accurate reconstructions of scenes from unstructured image collections taken from the internet for various landmarks.