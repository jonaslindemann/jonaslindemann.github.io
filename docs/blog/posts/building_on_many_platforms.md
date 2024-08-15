---
draft: false 
date: 2024-08-14
categories:
  - ForcePAD
  - C++
  - Backend
  - Raylib
  - CMake
  - vcpkg
---
# Building for many platforms is hard

![](images/multiplatform.png)

Choosing the right libraries for your application is just one step in making it run on multiple operating systems. ForcePAD uses CMake to build the application on multiple platforms. CMake is an excellent tool that generates build files for each platform. On Windows it generates Visual Studio solution files. On other platform it generates make files. I started implementing most of the tooling for ForcePAD on Windows thinking that most of this can be easily transferred to macOS. This what not that easy. I will go through the setup in the following sections.

<!-- more -->

## Windows and dependencies

The biggest issue with building applications on Windows is the dependencies that on many other operating systems such as Linux are easily installable using a package manager. Until recently there was no real packages manager available on Windows, requiring you to download, build and install dependencies manually. Today there are several package managers for C++ that can solve this problem, such as Conan and Vcpkg. For ForcePAD I chose to use Vcpkg as it has worked well for other projects. To use Vcpkg you provide a **vcpkg.json** file in your root build directory. In this file you specify the dependencies you have in your project. An example of this is shown below:

```json
{
    "name": "forcepad",
    "version": "0",
    "dependencies":
    [
        {
            "name": "raylib",
            "platform": "windows"
        },
        {
            "name": "opengl",
            "platform": "windows"
        },    
        {
            "name": "eigen3",
            "platform": "windows"
        },
        "stb",
        {
            "name": "imgui",
            "features": ["docking-experimental"]
        },
        {
            "name": "glfw3",
            "platform": "windows"
        }
    ]
}
```

The important stuff is contained in the **dependencies** section. Here you list the packages you require. The **platform** attributes indicates for which platform this dependency is required. It is optional, but I will come back to this later. To build the dependencies you type:

```cmd
C:\...\>vcpkg install
```

This will build the dependencies in your current directory. By default build libraries will be placed in the **vcpkg_installed** directory. 

To make sure these libraries are found by CMake we need to define a **CMakePresets.json** file where we specify where Vcpkg can be found for the different platforms. The preset file for ForcePAD is shown below:

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "default",
      "toolchainFile": "e:/vcpkg/scripts/buildsystems/vcpkg.cmake"
    },
    {
      "name": "linux",
      "toolchainFile": "/home/bmjl/vcpkg/scripts/buildsystems/vcpkg.cmake"
    },
    {
      "name": "macos",
      "toolchainFile": "/Users/lindemann/vcpkg/scripts/buildsystems/vcpkg.cmake"
    }
  ]
}
```

## A CMakeLists.txt file for all platforms

The nice thing using a package manager is that the actual **CMakeLists.txt** file doesn't have to have explicit paths for different libraries. Normal **find_package(...)** commands can be called and corresponding paths will be found. The commands used to find the packages are the following:

```CMake
find_package(raylib CONFIG REQUIRED)
find_package(Eigen3 CONFIG REQUIRED)
find_package(imgui CONFIG REQUIRED)
find_package(OpenGL REQUIRED)
find_package(Stb REQUIRED) 
find_package(glfw3 CONFIG REQUIRED)
```

## Configuring and building with CMake

To configure a debug build of ForcePAD using the **default** preset CMake is called with the following command line:

```cmd
C:\...\>cmake -B build-debug -DCMAKE_BUILD_TYPE=Debug --preset default
```

This configures the ForcePAD debug build in the **build-debug** directory. Building ForcePAD can now be done using the following command:

```cmd
C:\...\>cmake --build build-debug --config Debug -- /m
```

It is of course possible to use Visual Studio solution files contained in the **build-debug** directory to edit and build ForcePAD.

## Multiplatform is still hard

At this point I was very happy to have build system that at least in theory should work on both Windows and macOS. I copied over all my source files to my newly aquired Mac mini. Installed all development tools and MacBrew. 

I ran the same commands and Vcpkg started to build the dependencies, but the compiler and linker errors where plenty. I realised that using the Apple provided g++ compiler was not a good choice, so I switched to the brew provided compilers g++-13/gcc-13. This required me to make sure that they where used instead of the Apple-compilers. Using the following statements I set the required environment variables so that CMake and Vcpkg picked up the right compilers:

```bash
export CXX=g++-13
export CC=gcc-13
```

This reduced some of the compiler errors, but I still got a lot of linker errors with Vcpkg build libraries. Then I had an idea. Perhaps I should use brew to provide some of the packages instead. I added the *platform* directive in the vcpkg.json file and I told vcpkg only to build raylib, opengl, eigen3 and glfw3 on Windows. This make a lot of errors go away. However I got stuck on missing system libraries from Apple such as CoreGraphics and Cocoa. I modified my CMakeLists.txt for the main ForcePAD executable to the following:

```CMake
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    if (APPLE)
        add_executable(forcepad3d main.cpp forcepad_window.cpp forcepad_window.h)
        target_include_directories(forcepad3d PRIVATE ${gui_INCLUDE_DIRS} ${raylib_INCLUDE_DIRS})
        target_link_libraries(forcepad3d PRIVATE "-framework CoreGraphics" "-framework Cocoa" "-framework IOKit" "-framework OpenGL" "-framework CoreFoundation" rlimguid guid graphicsd ${raylib_LIBRARIES} OpenGL::GL imgui::imgui glfw)
    else()
        ...
```

These changes made ForcePAD build on macOS and worked on first startup. To make configuration a bit I created a **build.cmd** on Windows and a **build.sh** on macOS to automate some of the steps.

## Conclusions

Building a multiplatform application is hard. Raylib made it a bit easier on the source level, but still requires dependencies on the system side. It also worries me a bit that Raylib relies on OpenGL, which is kind of unsupported on macOS, but I suspect the Raylib developers could do some port of the **rlgl** to metal or other libraries.





