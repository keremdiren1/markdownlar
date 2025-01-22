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

This part contains a function that draws the axes on the canvas. The color of the axes is gray.  
The first two lines decide the width and color of the line (width is 1 and color is gray). The remaining 3 bundles of code all do similar things; they all draw a line in either x, y, or z axis.

``` javascript
    // Draw shapes on the canvas
    const drawShapes = () => {
      shapes.forEach((shape) => {
        if (shape.shape === "circle") {
          const { radius } = shape.parameters;

         
          const plane = shape.plane;

          let [x, y, z] = shape.coordinates;

        if (plane === "XYConstructionPlane") {
          [x, y, z] = shape.coordinates; // Reassign without redeclaring
        } else if (plane === "XZConstructionPlane") {
          [x, z, y] = shape.coordinates; // Reassign order for XZ plane
        } else if (plane === "YZConstructionPlane") {
          [y, z, x] = shape.coordinates; // Reassign order for YZ plane
        }



          const steps = 100; // Number of steps for circle approximation
          const points = [];

          for (let i = 0; i < steps; i++) {
            const angle = (i / steps) * Math.PI * 2;
            let px, py, pz;

            if (plane === "XYConstructionPlane") {
              px = x + radius * Math.cos(angle);
              py = y + radius * Math.sin(angle);
              pz = z;
            } else if (plane === "XZConstructionPlane") {
              px = x + radius * Math.cos(angle);
              py = y;
              pz = z + radius * Math.sin(angle);
            } else if (plane === "YZConstructionPlane") {
              px = x;
              py = y + radius * Math.cos(angle);
              pz = z + radius * Math.sin(angle);
            }

            points.push(transform3DTo2D(px, py, pz));
          }

          ctx.strokeStyle = "blue";
          ctx.lineWidth = 2;

          ctx.beginPath();
          points.forEach((point, index) => {
            if (index === 0) {
              ctx.moveTo(point.x, point.y);
            } else {
              ctx.lineTo(point.x, point.y);
            }
          });
          ctx.closePath();
          ctx.stroke();
        } else if (shape.shape === "rectangle") {
          const { width, height } = shape.parameters;
          const [x, y, z] = shape.coordinates;
          const plane = shape.plane;

          const corners = [];

          if (plane === "XYConstructionPlane") {
            corners.push(
              transform3DTo2D(x, y, z),
              transform3DTo2D(x + width, y, z),
              transform3DTo2D(x + width, y + height, z),
              transform3DTo2D(x, y + height, z)
            );
          } else if (plane === "XZConstructionPlane") {
            corners.push(
              transform3DTo2D(x, y, z),
              transform3DTo2D(x + width, y, z),
              transform3DTo2D(x + width, y, z + height),
              transform3DTo2D(x, y, z + height)
            );
          } else if (plane === "YZConstructionPlane") {
            corners.push(
              transform3DTo2D(x, y, z),
              transform3DTo2D(x, y + width, z),
              transform3DTo2D(x, y + width, z + height),
              transform3DTo2D(x, y, z + height)
            );
          }

          ctx.strokeStyle = "green";
          ctx.lineWidth = 2;

          ctx.beginPath();
          corners.forEach((corner, index) => {
            if (index === 0) {
              ctx.moveTo(corner.x, corner.y);
            } else {
              ctx.lineTo(corner.x, corner.y);
            }
          });
          ctx.closePath();
          ctx.stroke();
        }
      });
    };
```
