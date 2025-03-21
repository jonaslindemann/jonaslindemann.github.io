---
draft: false 
date: 2025-03-21
categories:
  - OpenGL
  - C++
  - Ivf++
  - Graphics
  - AI
  - Claude
---

# Experimenting with AI and Mechanics applications

![](images/ai_and_mechanics.png)

When working on the next version of ObjectiveFrame with improved scripting features I got the idea to see how good an AI agent would be in creating a mechanical structure using the built-in scripting language (Chaiscript) in ObjectiveFrame. I hade some initial sessions with Claude which confirmed that it was possible to create scripts from textual prompts and run them in the ObjectiveFrame's new scripting environment. This lead me to my next idea of adding an interface to an AI agent directly in the ObjectiveFrame application. The AI agent would then be able to create mechanical structures based on the user's prompt. In the following sections I will describe the process I went through to implement this feature.

<!-- more -->

## First conversations with Claude

To try out my idea of generating structures from prompts I gave Claude the following prompt:

!!! question 

    I have an 3D beam/bar analysis application that provide the following functions for creating geometry (implemented in chaiscript):

        vfem::Node *addNode(double x, double y, double z);
        vfem::Beam *addBeam(int i0, int i1);
        size_t nodeCount();
        void nodePos(vfem::Node *node, double &x, double &y, double &z);
        void nodePosAt(int i, double &x, double &y, double &z);
        void updateNodePos(vfem::Node *node, double x, double y, double z);
        void updateNodePosAt(int i, double x, double y, double z);
        vfem::Node *nodeAt(int i);
        bool isNodeSelected(vfem::Node *node);
        bool isNodeSelectedAt(int i);
        double randFloat(double min, double max);
        int randInt(int min, int max);
        void randSeed();

    Using this information can you create a chaiscript from the following prompt:

    Create a beam structure for a building 10 x 10 m with a angled roof. Add stabilisation structures as well.

In the prompt I described my scripting interface and its functions. I also told Claude that I was using the scripting language Chaiscript. I then asked Claude to generate a script that would create a beam structure for a building with a 10 x 10 m base and an angled roof. I also asked Claude to add stabilisation structures to the building. The response I got was the following:

!!! answer

    I'll create a ChaiScript that generates a beam structure for a 10m x 10m building with an angled roof and stabilization structures.Building Structure with Angled Roof and StabilizationClick to open codeTap to openThe script I've created generates a 10m x 10m building with an angled roof and comprehensive stabilization structures. Here's how it works:

    Building Dimensions:

    10m x 10m footprint
    4m wall height
    6m peak roof height (2m above the walls)

    Main Structure:

    Creates a rectangular base with four corner columns
    Forms a simple gable roof with a central ridge beam
    Includes top wall beams connecting the columns

    Stabilization Elements:

    X-bracing on all four walls for lateral stability
    Additional midpoint supports along each wall
    Ridge beam with supporting connections to all walls
    Purlins (horizontal roof supports) to reinforce the roof structure

    Node Arrangement:

    Ground corners are placed in a counter-clockwise pattern
    Wall tops align with the ground corners
    Roof peak forms along a central ridge line

    The script uses your provided ChaiScript functions to create nodes at specific coordinates and connect them with beams. It includes helper functions to simplify node creation and tracking.
    Would you like me to modify any aspects of this building structure? For example, I could adjust the roof angle, add more stabilization elements, or change the building proportions.

To my big surpris Claude actually generated a script that ran directly in ObjectiveFrame and created a beam structure for a building with a 10 x 10 m base and an angled roof. The script also added stabilisation structures to the building. The code was well structured and used convenience functions to simplify the creation of nodes and beams. Some of the code is shown below:

```cpp
// Building Structure Generator
// Creates a 10m x 10m building with angled roof and stabilization elements

// Define building dimensions
var width = 10.0;
var length = 10.0;
var wallHeight = 4.0;
var roofPeakHeight = 6.0;  // Peak of the roof is 2m above the wall tops

// Function to create node and return its index
def createNodeAndGetIndex(x, y, z) {
    var node = addNode(x, y, z);
    return nodeCount() - 1;  // Return the index of the newly created node
}

// Create building nodes - ground floor corners (counter-clockwise from origin)
print("Creating building structure...");

// Ground floor corners
var n0 = createNodeAndGetIndex(0.0, 0.0, 0.0);         // Front left
var n1 = createNodeAndGetIndex(width, 0.0, 0.0);       // Front right
var n2 = createNodeAndGetIndex(width, length, 0.0);    // Back right
var n3 = createNodeAndGetIndex(0.0, length, 0.0);      // Back left

// Top of walls (directly above ground floor corners)
var n4 = createNodeAndGetIndex(0.0, 0.0, wallHeight);         // Front left top
var n5 = createNodeAndGetIndex(width, 0.0, wallHeight);       // Front right top
var n6 = createNodeAndGetIndex(width, length, wallHeight);    // Back right top
var n7 = createNodeAndGetIndex(0.0, length, wallHeight);      // Back left top

// Roof peak nodes
var n8 = createNodeAndGetIndex(width/2, 0.0, roofPeakHeight);     // Front center peak
var n9 = createNodeAndGetIndex(width/2, length, roofPeakHeight);  // Back center peak

// Create center ridge beam
var ridge = createNodeAndGetIndex(width/2, length/2, roofPeakHeight);

// Create vertical beams (columns)
print("Creating columns...");
addBeam(n0, n4);  // Front left column
addBeam(n1, n5);  // Front right column
addBeam(n2, n6);  // Back right column
addBeam(n3, n7);  // Back left column

// Create ground floor beams
print("Creating ground floor frame...");
addBeam(n0, n1);  // Front ground beam
addBeam(n1, n2);  // Right ground beam
addBeam(n2, n3);  // Back ground beam
addBeam(n3, n0);  // Left ground beam

// ... more code ...
```

Running the code in ObjectiveFrame created the following structure:

![](images/ai_objframe_01.png)

Obviously the structure was created in the XY-plane, but the ground plane in ObjectiveFrame is the XZ-plane. I told Claude that I wanted the structure to be created in the XZ-plane instead. Claude then generated a new script that created the structure in the XZ-plane. The code was similar to the previous one but with the coordinates adjusted to the XZ-plane. The code was well structured and used convenience functions to simplify the creation of nodes and beams. Some of the code is shown below:
