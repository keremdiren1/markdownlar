# Markdown

parametrix test

## Components

### CoordinatePlane.js

``` javascript
import React, { useRef, useEffect } from "react";
```

This line imports required code from the react library.

``` javascript
const CoordinatePlane = ({ shapes }) => {
```

This line defines a functional react component.  
A functional react component is like a function but it doesn't require a class etc. and it returns a react element.

``` javascript
  const canvasRef = useRef(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");

    // Canvas size and configuration
    canvas.width = 800;
    canvas.height = 700;

    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const unit = 16; // Scaling unit for smaller shapes
```

The first line in this part creates a reference to the `<canvas>` DOM element, which gives us direct access to draw.  

``` javascript
  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
```

Inside useEffect, the context ctx is obtained (using the first two lines), which provides us some drawing methods.  

``` javascript
    // Canvas size and configuration
    canvas.width = 800;
    canvas.height = 700;

    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const unit = 16; // Scaling unit for smaller shapes
```

In this part, the width, height, origin coordinates, and scale of the canvas is set.
