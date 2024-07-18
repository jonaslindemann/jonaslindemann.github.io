---
draft: false 
date: 2024-07-19
categories:
  - ForcePAD
  - C++
  - Multiplatform
---
# Thoughts the ForcePAD application

ForcePAD was developed as part of my PhD thesis almost 20 years ago. The application was developed in C++ using OpenGL for graphics and FLTK for the user interface. At the time, it was state-of-the-art and compiled on all platforms, such as Linux, Windows, and SGI Irix 6.5(!). The application was used extensively in teaching and is still used today.

<!-- more -->

The application has been fixed and updated over the years an is available in the Windows app store. However, it is getting harder and harder to compile on both Windows and MacOS due to OpenGL being deprecated on the Mac and being complicated to maintain on Windows. The application would be ideal for running on an iPad, but as it uses OpenGL 1.1, which is not available on the iPad, this is also a dead end. 

Reflecting on my recent experience with the partial modernization of the older application, ObjectiveFrame, I see a promising path for the ForcePAD application. While the 3D graphics remain a challenge due to the reliance on a large scene graph library using OpenGL 1.1, the potential for modernization is clear. 

As the ForcePAD application uses a much simpler 2D graphics model, I have decided it is time for a blank sheet instead of a partial rewrite. This also gives me an opportunity to test some new ideas for the application:

 * Layers for conceptual sketching, 
 * Blending
 * Background speculative computations
 * Immediate-mode user interfaces using ImGUI 
 * Upgraded multithreaded computational code using the Eigen C++ library. 

I will be publishing my development journey regularly on an upcoming blog. I will leave you with a short video with the user interface in its current state.

<iframe width="560" height="315" src="https://www.youtube.com/embed/L7gTLw8KJEk?si=S8HhaJxOVs4D85rf" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>