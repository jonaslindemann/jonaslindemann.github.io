---
draft: true 
date: 2024-07-18
categories:
  - Introduction
---
# My blog post

Hello, world!

```cpp
void ForcePadWindow::onDraw()
{
    ClearBackground(WHITE);

    m_drawing->currentLayer()->beginDraw();

    // m_renderTexture->beginDraw();

    if (currentMouseButton() == gui::MouseButton::LEFT_BUTTON)
    {
        if (m_drawingMode == DrawingMode::Brush)
            m_brush->apply(mouseX(), this->monitorHeight() - mouseY());
        else if (m_drawingMode == DrawingMode::Eraser)
            m_eraser->apply(mouseX(), this->monitorHeight() - mouseY());
    }

    m_drawing->currentLayer()->endDraw();
```

<!-- more -->