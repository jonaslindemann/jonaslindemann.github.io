---
draft: false 
date: 2024-08-21
categories:
  - ForcePAD
  - C++
  - Concept
  - Design
---

# The conceptual model of ForcePAD 3

![ForcePAD Concept](images/forcepad_concept.png)

When redesigning an application, there is an opportunity to rethink the existing design and make improvements. In the case of ForcePAD 2, it was a simple pixel-based drawing application with a single layer for drawing. Users could add loads and boundary conditions on top of the drawing, and results were displayed on top of the image layer.

With ForcePAD 3, I aim to introduce a layered model that allows users to add multiple drawing layers on top of each other, similar to how a designer or architect uses sketch paper. These layers can be hidden and made transparent, providing more flexibility when sketching. In the following sections, I will provide a detailed description of the model and the main classes that implement it.

<!-- more -->

## A multi layered approach

In ForcePAD 3, I aim to optimize drawing routines by leveraging the GPU more than in ForcePAD 2. To implement the layered model, I have chosen to utilize the render-to-texture capabilities in RayLib. This approach allows for easy implementation of multiple layers. The following code demonstrates the basic idea:

```cpp
RenderTexture2D texture;

...

void draw()
{
    ...

    BeginTextureMode(texture);

    DrawEllipse(...)

    EndTextureMode()

    DrawTexture(texture, x, y, tint);
    
    ...
}
```

By rendering to a texture, each layer can be managed independently, providing flexibility and improved performance.

```cpp
RenderTexture2D texture;

...

void draw()
{
    ...

    BeginTextureMode(texture);

    DrawEllipse(...)

    EndTextureMode()

    DrawTexture(texture, x, y, tint);
    
    ...
}
```

To abstract this model into something that can implement the layered model in ForcePAD 3, I decided to create a main **Drawing** class that contains multiple **Layer** instances, each managing a render texture for a specific layer. The handling of render textures is implemented in the **RaylibRenderTexture** class.

The **Drawing** class has two methods, **beginDraw()** and **endDraw()**, which call the corresponding methods of the **Layer** class. This simplifies the main drawing routine in the application class:

```cpp
void ForcePadWindow::onDraw()
{
    ClearBackground(WHITE);

    m_drawing->beginDraw();

    // Pixel drawing operations to current layer.

    m_drawing->endDraw();

    // Draw all textures

    m_drawing->draw();
}
```

This abstraction allows for easy management of multiple layers and improves the overall organization of the code.

```cpp
void ForcePadWindow::onDraw()
{
    ClearBackground(WHITE);

    // Initiate render to current layer and texture

    m_drawing->beginDraw();

    // Pixel drawing operations to current layer.

    m_drawing->endDraw();

    m_drawing->updateMouse(mouseX(), mouseY());

    // Draw all textures

    m_drawing->draw();
}
```

The **Drawing** **draw()** method is also relatively small.

```cpp
void Drawing::draw()
{
    ...

    for (auto &layer : m_layers)
    {
        if (layer->visible())
        {
            layer->draw();
        }
    }

    ...
}
```

I was initially concerned about the performance impact of using screen-sized render textures. However, after testing it on various hardware, I was pleasantly surprised to find that it performs remarkably well. The application consistently maintains a smooth frame rate of 60 frames per second, even when multiple layers are used.

## Adding vector drawing 

To improve precision when sketching, I decided to implement vector-based drawing in ForcePAD. RayLib provides many built-in functions for drawing basic shapes, making it a suitable choice as a base for drawing the shapes. To achieve this, I created an abstract class **Shape** to store generic attributes such as fill color, line color, and border thickness for vector shapes. Additionally, I implemented specific **Shape** classes like **Ellipse**, **Rectangle**, and **Line** to implement concreate drawing of the shapes.

To avoid implementing a separate vector drawing mechanism, each **Layer** class maintains a set of shapes that are drawn on top of the render texture. As a result, the **draw()** method of the layer becomes responsible for rendering these shapes.

```cpp
void graphics::Layer::draw()
{
    m_renderTexture->draw(0, 0, m_tint);

    for (auto &shape : m_shapes)
    {
        shape->draw();
    }
}
```

## A first verion of the ForcePAD model

The **Drawing** class will serve as the foundation for implementing the ForcePAD model. It will be a layered model consisting of the following layers:

1. Drawing layers: Manages pixel and vector shapes.
2. Result layer: Contains the results from the simulated model.
3. Force and constraints layer: Draws and maintains forces and constraints for the model.

The class will handle interactions with shapes, pixels, forces, and constraints. It will also implement features such as hovering and selection. Additionally, the **Drawing** class will serve as the main data model for ForcePAD, allowing for serialization of the model to disk. The plan is to use JSON as the primary file format for ForcePAD, enabling easy reading and writing of models by other applications. The implementation of serialization will be done in a separate class.

I am satisfied with the current model and it has been successfully implemented in the latest version of ForcePAD 3 on GitHub. No significant modifications have been necessary for the underlying model as of yet.

