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

``` javascript
    // Function to convert 3D coordinates to 2D screen space
    const transform3DTo2D = (x, y, z) => {
      const angleX = Math.PI / 6; // Rotation angle for X
      const angleZ = Math.PI / 6; // Rotation angle for Z

      const transformedX = x * Math.cos(angleZ) - z * Math.sin(angleZ);
      const transformedY =
        y * Math.cos(angleX) - (x * Math.sin(angleZ) + z * Math.cos(angleZ)) * Math.sin(angleX);

      return {
        x: centerX + transformedX * unit,
        y: centerY - transformedY * unit,
      };
    };
```

This part contains a function which uses math to convert the 3D coordinates to 2D plane.

``` javascript
    // Draw axes on the canvas
    const drawAxes = () => {
      ctx.strokeStyle = "gray"; // Neutral color for axes
      ctx.lineWidth = 1;

      // X-axis
      ctx.beginPath();
      const xStart = transform3DTo2D(-64, 0, 0);
      const xEnd = transform3DTo2D(64, 0, 0);
      ctx.moveTo(xStart.x, xStart.y);
      ctx.lineTo(xEnd.x, xEnd.y);
      ctx.stroke();

      // Y-axis
      ctx.beginPath();
      const yStart = transform3DTo2D(0, -64, 0);
      const yEnd = transform3DTo2D(0, 64, 0);
      ctx.moveTo(yStart.x, yStart.y);
      ctx.lineTo(yEnd.x, yEnd.y);
      ctx.stroke();

      // Z-axis
      ctx.beginPath();
      const zStart = transform3DTo2D(0, 0, -64);
      const zEnd = transform3DTo2D(0, 0, 64);
      ctx.moveTo(zStart.x, zStart.y);
      ctx.lineTo(zEnd.x, zEnd.y);
      ctx.stroke();
    };
```

This part contains functions for
