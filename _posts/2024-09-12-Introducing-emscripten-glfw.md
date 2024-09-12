---
layout: post
title: "Introducing emscripten-glfw (aka contrib.glfw3)"
category: webassembly
tags:
 - emscripten
 - webassembly
 - wasm
 - glfw
---

### Introduction

[GLFW](https://www.glfw.org/) is an Open Source, multi-platform library that provides a simple API for 
creating windows, contexts, receiving input and events.
The primary implementation focuses on the desktop platforms.

With the Web and the browser platform taking over, and new technologies like [WebAssembly](https://webassembly.org/),
there is a strong desire to be able to write or port applications to this new paradigm.

[`emscripten-glfw`](https://github.com/pongasoft/emscripten-glfw) is an implementation of the GLFW API with the primary goal of implementing as many features as possible in the context of a web browser.

`emscripten-glfw` is now an [Emscripten port]({{site.url}}{% post_url 2024-02-19-the-power-of-emscripten-ports %}),
one that is easily available via the (`emcc`) option `--use-port=contrib.glfw3`.

To get a taste of how powerful this is, you can check out the [demo](https://pongasoft.github.io/emscripten-glfw/test/demo/main.html) which runs in any browser supporting OpenGL (all browsers should work).
Or [WebGPU Shader Toy](https://pongasoft.github.io/webgpu-shader-toy), which runs in a Web Browser supporting WebGPU
(Chrome or Edge as of this writing).

### History

Emscripten has a GLFW built-in implementation. It was implemented back in 2013 in pure JavaScript.
It currently states that it implements the `3.2.1` version of the API released in 2016
(as returned by `glfwGetVersion`).
It is not well maintained and lacks support for more recent additions like the Gamepad API.
It does not comply with the error handling paradigm of GLFW (`glfwSetErrorCallback` stores the function
pointer but never uses it and some APIs throw an exception when called (like `glfwGetTimerFrequency`)).

After doing some work on this JavaScript implementation (adding [Hi DPI/4K support](https://github.com/emscripten-core/emscripten/pull/20584)),
and adding [contrib port support](https://github.com/emscripten-core/emscripten/pull/21214) to Emscripten, it became clear to me
that a clean slate approach to implementing GLFW3 was required (and [suggested](https://github.com/emscripten-core/emscripten/pull/20584#issuecomment-1829957378)).

Fast-forward to today, and my implementation is not only available, but easily usable as an Emscripten port.
It can easily be integrated in any application wanting to use GLFW in the browser.

Since ImGui [v1.91.0](https://github.com/ocornut/imgui/releases/tag/v1.91.0),
ImGui can be configured to use this port, allowing full gamepad and clipboard support amongst many other advantages.

### Primary Features

For more details about these features, check the [documentation](https://github.com/pongasoft/emscripten-glfw/blob/master/docs/Usage.md). For a comparison with the built-in implementation, check the [comparison](https://github.com/pongasoft/emscripten-glfw/blob/master/docs/Comparison.md) page.

#### Multiple windows

In the context of the browser, a (GLFW) window is associated with an HTML canvas.
`emscripten-glfw` supports any number of windows.

Example 1: the [demo](https://pongasoft.github.io/emscripten-glfw/test/demo/main.html) uses 2 windows 

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/multi-window-1.gif" alt="2 windows (demo)" border="0" width="100%" style="">
  <figcaption>The right canvas/window is resizable like a desktop window using the handle</figcaption>
</figure>

Example 2: [WebGPU Shader Toy](https://pongasoft.github.io/webgpu-shader-toy) uses 2 windows as well

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/multi-window-2.png" alt="2 windows (WebGPU Shader Toy)" width="100%" border="0">
  <figcaption>Both windows are individually resizable via their respective handle</figcaption>
</figure>

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/multi-window-2.gif" alt="2 windows (WebGPU Shader Toy)" border="0" width="100%" >
  <figcaption>Or via the browser window</figcaption>
</figure>

<div class="info"><code>emscripten-glfw</code> provides a convenient API to make the canvas resizable (<code>emscripten::glfw3::MakeCanvasResizable</code>)</div>

#### Mouse and keyboard

The library fully supports the mouse and keyboard, including sticky button behavior (resp. sticky key behavior).

In addition, when the mouse is being held, and it moves outside the browser window, it continues to work, thus making moving a scrollbar natural and uninterrupted.

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/scrollbar.gif" alt="Mouse works outside the browser" border="0">
  <figcaption>Moving the scrollbar when the mouse is outside the browser</figcaption>
</figure>

#### Gamepad support

`emscripten-glfw` implements the full Gamepad API, thus making dealing with joysticks consistent across platforms.

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/standard_gamepad.svg" alt="The Gamepad" border="0">
  <figcaption>The standard gamepad</figcaption>
</figure>

#### Fullscreen and Hi DPI

The library has full support for fullscreen (with resizing when requested) and Hi DPI for 4K displays.
Both features are demonstrated in this [example](https://pongasoft.github.io/emscripten-glfw/examples/example_hi_dpi/main.html)
(using `Ctrl + H` to toggle between Hi DPI and Low DPI, and `Ctrl + F` to go fullscreen).

#### Clipboard - Copy/Paste

Dealing with the outside clipboard is a bit tricky in a browser environment due to security reasons.
This implementation supports both copy and paste with the desktop clipboard as long as the proper
shortcuts are being used.

<figure class="image img-responsive img-shadow">
  <img src="{{ site.url }}/resource/images/2024-09-12/copy-paste.gif" alt="Copy/Paste" border="0">
  <figcaption>Copy/Paste with an external application (left is WebGPU Shader Toy / right is TextMate)</figcaption>
</figure>

### Other Features

In addition to the previous features listed above, `emscripten-glfw` implements a host of other functionalities,
not implemented by the built-in implementation:

* Cursors (all GLFW cursors are supported)
* Window size constraints (size limits and aspect ratio)
* Window opacity
* Window status (Focused / Hovered / Position)
* Timer
* Error handling

### Give it a try

This has been a very fun project to work on. I hope this introduction has been helpful.
Feel free to give it a try for yourself and provide feedback or questions in the [GitHub project](https://github.com/pongasoft/emscripten-glfw) issues tracker.

If you want to integrate in your project, Emscripten offers a straightforward and convenient syntax:

```sh
emcc --use-port=contrib.glfw3 main.cpp -o /tmp/index.js
```

You can follow news and announcements on the [pongasoft website](https://pongasoft.com/news.html) (also available as an [RSS feed](https://pongasoft.com/atom.xml)).