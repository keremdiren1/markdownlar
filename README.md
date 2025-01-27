# Markdown

parametrix test

## Components

### CoordinatePlane.js

``` javascript
import React, { useRef, useEffect } from "react";

const CoordinatePlane = ({ shapes }) => {
  const canvasRef = useRef(null);

  // Constants for canvas configuration
  const CANVAS_WIDTH = 800;
  const CANVAS_HEIGHT = 700;
  const UNIT = 16;
  const STEPS = 100; // For smooth curves

  // Shape colors
  const COLORS = {
    circle: "blue",
    rectangle: "green",
    triangle: "red",
    ellipse: "purple",
    axes: "gray"
  };

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");

    // Set canvas size
    canvas.width = CANVAS_WIDTH;
    canvas.height = CANVAS_HEIGHT;

    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;

    // 3D to 2D transformation function
    const transform3DTo2D = (x, y, z) => {
      const angleX = Math.PI / 6;
      const angleZ = Math.PI / 6;

      const transformedX = x * Math.cos(angleZ) - z * Math.sin(angleZ);
      const transformedY =
        y * Math.cos(angleX) - 
        (x * Math.sin(angleZ) + z * Math.cos(angleZ)) * Math.sin(angleX);

      return {
        x: centerX + transformedX * UNIT,
        y: centerY - transformedY * UNIT,
      };
    };

    // Draw coordinate system axes
    const drawAxes = () => {
      ctx.strokeStyle = COLORS.axes;
      ctx.lineWidth = 1;

      // Draw X axis
      const xStart = transform3DTo2D(-64, 0, 0);
      const xEnd = transform3DTo2D(64, 0, 0);
      ctx.beginPath();
      ctx.moveTo(xStart.x, xStart.y);
      ctx.lineTo(xEnd.x, xEnd.y);
      ctx.stroke();

      // Draw Y axis
      const yStart = transform3DTo2D(0, -64, 0);
      const yEnd = transform3DTo2D(0, 64, 0);
      ctx.beginPath();
      ctx.moveTo(yStart.x, yStart.y);
      ctx.lineTo(yEnd.x, yEnd.y);
      ctx.stroke();

      // Draw Z axis
      const zStart = transform3DTo2D(0, 0, -64);
      const zEnd = transform3DTo2D(0, 0, 64);
      ctx.beginPath();
      ctx.moveTo(zStart.x, zStart.y);
      ctx.lineTo(zEnd.x, zEnd.y);
      ctx.stroke();
    };

    // Shape drawing functions
    const drawCircle = (shape) => {
      const { radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y + radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y;
            pz = z + radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + radius * Math.cos(angle);
            pz = z + radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.circle);
    };

    const drawRectangle = (shape) => {
      const { width, height } = shape.parameters;
      const [x, y, z] = shape.coordinates;
      const corners = [];

      switch (shape.plane) {
        case "XYConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y + height, z),
            transform3DTo2D(x, y + height, z)
          );
          break;
        case "XZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
        case "YZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + width, z),
            transform3DTo2D(x, y + width, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
      }

      drawPath(corners, COLORS.rectangle);
    };

    const drawTriangle = (shape) => {
      const { side1, side2, side3 } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      
      // Calculate angle using cosine law
      const angleA = Math.acos(
        (side2 * side2 + side3 * side3 - side1 * side1) / 
        (2 * side2 * side3)
      );

      const points = [];
      switch (shape.plane) {
        case "XYConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y + side3 * Math.sin(angleA),
              z
            )
          );
          break;
        case "XZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y,
              z + side3 * Math.sin(angleA)
            )
          );
          break;
        case "YZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + side2, z),
            transform3DTo2D(
              x,
              y + side3 * Math.cos(angleA),
              z + side3 * Math.sin(angleA)
            )
          );
          break;
      }

      drawPath(points, COLORS.triangle);
    };

    const drawEllipse = (shape) => {
      const { major_radius, minor_radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y + minor_radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y;
            pz = z + minor_radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + major_radius * Math.cos(angle);
            pz = z + minor_radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.ellipse);
    };

    // Helper function to draw paths
    const drawPath = (points, color) => {
      ctx.strokeStyle = color;
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
    };

    // Main drawing function
    const drawShapes = () => {
      shapes.forEach((shape) => {
        switch (shape.shape) {
          case "circle":
            drawCircle(shape);
            break;
          case "rectangle":
            drawRectangle(shape);
            break;
          case "triangle":
            drawTriangle(shape);
            break;
          case "ellipse":
            drawEllipse(shape);
            break;
        }
      });
    };

    // Clear canvas and draw everything
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawAxes();
    drawShapes();
  }, [shapes]);

  return (
    <canvas 
      ref={canvasRef} 
      style={{ 
        border: "2px solid black",
        margin: "20px",
      }} 
    />
  );
};

export default CoordinatePlane;
```

This part of the explanation (every text under CoordinatePlane.js) will be about the code above. Every explanation will be made with the text talking about the code block above it.

``` javascript
import React, { useRef, useEffect } from "react";
```

This line imports required code from the react library.

``` javascript
const CoordinatePlane = ({ shapes }) => {
```

This line defines a functional react component.  
A functional react component is like a function but it is simpler and it returns a react element.

``` javascript
  const canvasRef = useRef(null);
```

This line creates a reference to the `canvas` DOM element, which is important for us to put shapes on the screen.

``` javascript
  // Constants for canvas configuration
  const CANVAS_WIDTH = 800;
  const CANVAS_HEIGHT = 700;
  const UNIT = 16;
  const STEPS = 100; // For smooth curves
```

This code block creates some constants that will be used to set the canvas and shapes. The first 3 lines are about the canvas's width, height, and unit. The last line contains the number of points a shape like circle will have in a 2D plane so that after each point is connected, a circle (hectogon, to be more precise) will be created as close to the desired circle as possible.

``` javascript
  // Shape colors
  const COLORS = {
    circle: "blue",
    rectangle: "green",
    triangle: "red",
    ellipse: "purple",
    axes: "gray"
  };
```

The array above is created so that when a color is needed for a specific sitation, it could be accessed directly from this array. For example, every circle would be drawn blue, so the color written next to circle is blue.

```javascript
  useEffect(() => {
```

``` javascript
  }, [shapes]);
```

The line in the first code block above is used so that the code could be automatically re-run whenever shapes is changed. This condition can be seen in the second code block above, which is the end of the `useEffect(() => {`.

``` javascript
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
```

This code block creates ctx, which lets us create shapes in the canvas.

``` javascript
    // Set canvas size
    canvas.width = CANVAS_WIDTH;
    canvas.height = CANVAS_HEIGHT;

    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
```

This code block sets the width and height of the canvas to the constants created earlier. This code block also creates centerX and centerY, which represents the center coordinates of the canvas.

``` javascript
    // 3D to 2D transformation function
    const transform3DTo2D = (x, y, z) => {
      const angleX = Math.PI / 6;
      const angleZ = Math.PI / 6;

      const transformedX = x * Math.cos(angleZ) - z * Math.sin(angleZ);
      const transformedY =
        y * Math.cos(angleX) - 
        (x * Math.sin(angleZ) + z * Math.cos(angleZ)) * Math.sin(angleX);

      return {
        x: centerX + transformedX * UNIT,
        y: centerY - transformedY * UNIT,
      };
    };
```

This code block contains a function that turns 3D coordinates to 2D ones. We input x, y, and z coordinates, and it returns x and y coordinates depending on the coordinates the function was inputted.

``` javascript
    // Draw coordinate system axes
    const drawAxes = () => {
      ctx.strokeStyle = COLORS.axes;
      ctx.lineWidth = 1;

      // Draw X axis
      const xStart = transform3DTo2D(-64, 0, 0);
      const xEnd = transform3DTo2D(64, 0, 0);
      ctx.beginPath();
      ctx.moveTo(xStart.x, xStart.y);
      ctx.lineTo(xEnd.x, xEnd.y);
      ctx.stroke();

      // Draw Y axis
      const yStart = transform3DTo2D(0, -64, 0);
      const yEnd = transform3DTo2D(0, 64, 0);
      ctx.beginPath();
      ctx.moveTo(yStart.x, yStart.y);
      ctx.lineTo(yEnd.x, yEnd.y);
      ctx.stroke();

      // Draw Z axis
      const zStart = transform3DTo2D(0, 0, -64);
      const zEnd = transform3DTo2D(0, 0, 64);
      ctx.beginPath();
      ctx.moveTo(zStart.x, zStart.y);
      ctx.lineTo(zEnd.x, zEnd.y);
      ctx.stroke();
    };
```

The first 2 lines of this code block sets the color of the line that will be drawn to COLORS.axes, which is gray, and it also sets the width pf the line that will be drawn to 1. The remainder of the code draws the x, y, and z axes to the canvas.

``` javascript
    // Shape drawing functions
    const drawCircle = (shape) => {
      const { radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y + radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y;
            pz = z + radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + radius * Math.cos(angle);
            pz = z + radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.circle);
    };

    const drawRectangle = (shape) => {
      const { width, height } = shape.parameters;
      const [x, y, z] = shape.coordinates;
      const corners = [];

      switch (shape.plane) {
        case "XYConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y + height, z),
            transform3DTo2D(x, y + height, z)
          );
          break;
        case "XZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
        case "YZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + width, z),
            transform3DTo2D(x, y + width, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
      }

      drawPath(corners, COLORS.rectangle);
    };

    const drawTriangle = (shape) => {
      const { side1, side2, side3 } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      
      // Calculate angle using cosine law
      const angleA = Math.acos(
        (side2 * side2 + side3 * side3 - side1 * side1) / 
        (2 * side2 * side3)
      );

      const points = [];
      switch (shape.plane) {
        case "XYConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y + side3 * Math.sin(angleA),
              z
            )
          );
          break;
        case "XZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y,
              z + side3 * Math.sin(angleA)
            )
          );
          break;
        case "YZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + side2, z),
            transform3DTo2D(
              x,
              y + side3 * Math.cos(angleA),
              z + side3 * Math.sin(angleA)
            )
          );
          break;
      }

      drawPath(points, COLORS.triangle);
    };

    const drawEllipse = (shape) => {
      const { major_radius, minor_radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y + minor_radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y;
            pz = z + minor_radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + major_radius * Math.cos(angle);
            pz = z + minor_radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.ellipse);
    };

    // Helper function to draw paths
    const drawPath = (points, color) => {
      ctx.strokeStyle = color;
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
    };

    // Main drawing function
    const drawShapes = () => {
      shapes.forEach((shape) => {
        switch (shape.shape) {
          case "circle":
            drawCircle(shape);
            break;
          case "rectangle":
            drawRectangle(shape);
            break;
          case "triangle":
            drawTriangle(shape);
            break;
          case "ellipse":
            drawEllipse(shape);
            break;
        }
      });
    };
```

The code block above contains the code for drawing shapes. This will be further explained below.

``` javascript
    // Main drawing function
    const drawShapes = () => {
      shapes.forEach((shape) => {
        switch (shape.shape) {
          case "circle":
            drawCircle(shape);
            break;
          case "rectangle":
            drawRectangle(shape);
            break;
          case "triangle":
            drawTriangle(shape);
            break;
          case "ellipse":
            drawEllipse(shape);
            break;
        }
      });
    };
```

I will first start by explaining this function, which is the main drawing function. This function calls the appropriate function for each shape that's in shapes.

``` javascript
    const drawCircle = (shape) => {
      const { radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y + radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + radius * Math.cos(angle);
            py = y;
            pz = z + radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + radius * Math.cos(angle);
            pz = z + radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.circle);
    };
```

This code block creates a function to create 2D circle based on the inputted information about the shape. The first few lines set the radius of the shape to a constant named radius; they also set x, y, and z variables depending on the coordinates of the shape and create an empty points array, which will be used to contain the corner points of the shape (circles don't have corners, so the shape will be a polygon with 100 corners, which will look similar to the circle we desire).  
The for loop iterates as much as STEPS, which has a value of 100. Every value of i from the for loop is used to calculate which angle of the circle the current point (the point that is being determined at the current iteration) is at. For example, if the for loop is at the 2nd iteration, the angle of the point is (1/100) * 2 * pi, which is 1% of a maximum value of 2pi. The for loop also determines the variables of px, py, and pz, which are the coordinates of the current point. These points are determined depending on the current angle, radius, and the plane (the plane can be XY, XZ, or YZ plane). After the coordinate of the point is determined, the coordinate is added to the points array after being transformed to 2D coordinates. Basically, the for loop is used to determine 100 points on the given circle.  
After this, the points array and the COLORS.circle color (blue) is inputted into the drawPath function. I will be explaining the drawPath function after explaining the functions of the shapes.

``` javascript
    const drawRectangle = (shape) => {
      const { width, height } = shape.parameters;
      const [x, y, z] = shape.coordinates;
      const corners = [];

      switch (shape.plane) {
        case "XYConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y + height, z),
            transform3DTo2D(x, y + height, z)
          );
          break;
        case "XZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + width, y, z),
            transform3DTo2D(x + width, y, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
        case "YZConstructionPlane":
          corners.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + width, z),
            transform3DTo2D(x, y + width, z + height),
            transform3DTo2D(x, y, z + height)
          );
          break;
      }

      drawPath(corners, COLORS.rectangle);
    };
```

The code block above is used to create a function that's used to create a 2D rectangle. The function first determines the height and width of the rectangle, and then it finds the rectangle's coordinates and creates a corners array (this array is similar to the points array in the drawCircle function). After this, the program determines 4 points (1 for each corner) depending on the width, height, and the plane of the rectangle, which it turns from 3D to 2D before adding it to the corners array. At the end, the drawPath function is called by inputting the corners array and the COLORS.rectangle color.

``` javascript
    const drawTriangle = (shape) => {
      const { side1, side2, side3 } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      
      // Calculate angle using cosine law
      const angleA = Math.acos(
        (side2 * side2 + side3 * side3 - side1 * side1) / 
        (2 * side2 * side3)
      );

      const points = [];
      switch (shape.plane) {
        case "XYConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y + side3 * Math.sin(angleA),
              z
            )
          );
          break;
        case "XZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x + side2, y, z),
            transform3DTo2D(
              x + side3 * Math.cos(angleA),
              y,
              z + side3 * Math.sin(angleA)
            )
          );
          break;
        case "YZConstructionPlane":
          points.push(
            transform3DTo2D(x, y, z),
            transform3DTo2D(x, y + side2, z),
            transform3DTo2D(
              x,
              y + side3 * Math.cos(angleA),
              z + side3 * Math.sin(angleA)
            )
          );
          break;
      }

      drawPath(points, COLORS.triangle);
    };
```

The code block above is used to create a function which's purpose is to draw a 2D triangle. The code first destructures the side lengths and the coordinate of the triangle. Then, it calculates the angle by using the lengths it took. After this, the code creates a points array, which will be used to contain the coordinates of the corner points of the triangle. Finally, the coordinates of the corner points are calculated based on the plane, side lengths, and the angle determined before, which are then added to the points array (the coordinates are turned to 2D before being added, of course). Like the other shape drawing functions, the drawPath function is called to draw the triangle based on the points array and the COLORS.triangle color.

``` javascript
    const drawEllipse = (shape) => {
      const { major_radius, minor_radius } = shape.parameters;
      let [x, y, z] = shape.coordinates;
      const points = [];

      for (let i = 0; i < STEPS; i++) {
        const angle = (i / STEPS) * Math.PI * 2;
        let px, py, pz;

        switch (shape.plane) {
          case "XYConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y + minor_radius * Math.sin(angle);
            pz = z;
            break;
          case "XZConstructionPlane":
            px = x + major_radius * Math.cos(angle);
            py = y;
            pz = z + minor_radius * Math.sin(angle);
            break;
          case "YZConstructionPlane":
            px = x;
            py = y + major_radius * Math.cos(angle);
            pz = z + minor_radius * Math.sin(angle);
            break;
        }

        points.push(transform3DTo2D(px, py, pz));
      }

      drawPath(points, COLORS.ellipse);
    };
```

The code block above contains a function that is used to draw a 2D ellipse. First, the code takes the major radius, minor radius, and the coordinates from the ellipse. It then creates a points array, which will contain 100 coordinates of points placed in the shape of the desired 2D ellipse. After this, a for loop that iterates as much as STEPS, which has the value of 100, is created.  
This for loop first determines an angle based on the current value of i. It then calculates the x, y, and z coordinates of the current point (the point that is in the process of being determined in the current iteration) based on the plane, angle, and the minor and major radius of the ellipse. It then adds this coordinate to the points array after turning it into 2D. The for loop repeats this a 100 times, while only changing the current angle at each iteration (as I said before).  
After the for loop ends, the drawPath function is called to draw the ellipse based on the points array and the COLORS.ellipse color.

``` javascript
    // Helper function to draw paths
    const drawPath = (points, color) => {
      ctx.strokeStyle = color;
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
    };
```

I will now explain this function, which draws the shapes. This function first gets the color and points as an input. The color is inputted based on the shape that is being drawn (it's blue if it's a circle, for example). The points is the array of coordinates that was created before this function was called.  
This function first sets the color of the line that will be drawn to the color inputted and the line width to 2. Then, it creates a for each loop that creates lines to connect the points in the points array.

``` javascript
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawAxes();
    drawShapes();
```

We are now at the end of the explanation of CoordinatePlane.js. The code block above clears the current canvas and calls the methods named drawAxes and drawShapes, which draw the x, y, and z axes and the current shapes.  
I also want to say that this code returns the canvas it created with the canvas having a black border with 2px thickness.

### MainPage.js

``` javascript
import React from "react";
import { useNavigate } from "react-router-dom";

const MainPage = () => {
  const navigate = useNavigate();

  return React.createElement(
    "div",
    {
      style: {
        display: "flex",
        flexDirection: "column",
        justifyContent: "center",
        alignItems: "center",
        height: "100vh",
        backgroundColor: "black",
        color: "white",
        fontFamily: "'Press Start 2P'",
        textAlign: "center",
      },
    },
    // Title
    React.createElement(
      "h1",
      {
        style: {
          fontSize: "4rem",
          textTransform: "uppercase",
          marginBottom: "40px",
          letterSpacing: "2px",
        },
      },
      React.createElement("span", { style: { color: "#FFA500" } }, "PARAMETRI"),
      "X"
    ),
    // Start Button
    React.createElement(
      "button",
      {
        style: {
          fontSize: "1rem",
          textTransform: "uppercase",
          fontFamily: "'Press Start 2P'",
          backgroundColor: "black",
          color: "white",
          border: "2px solid white",
          padding: "10px 20px",
          cursor: "pointer",
          transition: "transform 0.2s, background-color 0.2s",
        },
        onClick: () => navigate("/split"), // Navigate to SplitSelection
        onMouseEnter: (e) => {
          e.target.style.backgroundColor = "#FFA500";
          e.target.style.color = "black";
          e.target.style.transform = "scale(1.1)";
        },
        onMouseLeave: (e) => {
          e.target.style.backgroundColor = "black";
          e.target.style.color = "white";
          e.target.style.transform = "scale(1)";
        },
      },
      "Start"
    ),
    // Blinking Text
    React.createElement(
      "p",
      {
        style: {
          marginTop: "30px",
          fontSize: "0.7rem",
          color: "#bbb",
          textTransform: "uppercase",
          fontFamily: "'Press Start 2P'",
          animation: "blink 2s infinite",
        },
      },
      "Press Enter to Start"
    )
  );
};

export default MainPage;
```

The text under MainPage.js will be about the code above.

``` javascript
import React from "react";
import { useNavigate } from "react-router-dom";
```

This code block imports the react library and useNavigate from reacter-router-dom.

``` javascript
const MainPage = () => {
```

This line creates a react functional component called MainPage.

``` javascript
  const navigate = useNavigate();
```

This line gives us the ability to navigate to different pages. For example, `navigate("/split")` makes the page navigate to SplitSelection, which is a different page.

``` javascript
  return React.createElement(
    "div",
    {
      style: {
        display: "flex",
        flexDirection: "column",
        justifyContent: "center",
        alignItems: "center",
        height: "100vh",
        backgroundColor: "black",
        color: "white",
        fontFamily: "'Press Start 2P'",
        textAlign: "center",
      },
    },
    // Title
    React.createElement(
      "h1",
      {
        style: {
          fontSize: "4rem",
          textTransform: "uppercase",
          marginBottom: "40px",
          letterSpacing: "2px",
        },
      },
      React.createElement("span", { style: { color: "#FFA500" } }, "PARAMETRI"),
      "X"
    ),
    // Start Button
    React.createElement(
      "button",
      {
        style: {
          fontSize: "1rem",
          textTransform: "uppercase",
          fontFamily: "'Press Start 2P'",
          backgroundColor: "black",
          color: "white",
          border: "2px solid white",
          padding: "10px 20px",
          cursor: "pointer",
          transition: "transform 0.2s, background-color 0.2s",
        },
        onClick: () => navigate("/split"), // Navigate to SplitSelection
        onMouseEnter: (e) => {
          e.target.style.backgroundColor = "#FFA500";
          e.target.style.color = "black";
          e.target.style.transform = "scale(1.1)";
        },
        onMouseLeave: (e) => {
          e.target.style.backgroundColor = "black";
          e.target.style.color = "white";
          e.target.style.transform = "scale(1)";
        },
      },
      "Start"
    ),
    // Blinking Text
    React.createElement(
      "p",
      {
        style: {
          marginTop: "30px",
          fontSize: "0.7rem",
          color: "#bbb",
          textTransform: "uppercase",
          fontFamily: "'Press Start 2P'",
          animation: "blink 2s infinite",
        },
      },
      "Press Enter to Start"
    )
  );
```

This part contains the react elements that will be returned by this functional react component. I will explain this code block in pieces.

``` javascript
  return React.createElement(
    "div",
    {
      style: {
        display: "flex",
        flexDirection: "column",
        justifyContent: "center",
        alignItems: "center",
        height: "100vh",
        backgroundColor: "black",
        color: "white",
        fontFamily: "'Press Start 2P'",
        textAlign: "center",
      },
    },
```

This code block creates a full page with a black background, centered white content, and a retro font.

``` javasctipt
    // Title
    React.createElement(
      "h1",
      {
        style: {
          fontSize: "4rem",
          textTransform: "uppercase",
          marginBottom: "40px",
          letterSpacing: "2px",
        },
      },
      React.createElement("span", { style: { color: "#FFA500" } }, "PARAMETRI"),
      "X"
    ),
```

This part creates a title which is as big as 4 rem units, is uppercase, and has "PARAMETRIX" written. The "X" in "PARAMETRIX" is white, while other text is orange.

``` javascript
    // Start Button
    React.createElement(
      "button",
      {
        style: {
          fontSize: "1rem",
          textTransform: "uppercase",
          fontFamily: "'Press Start 2P'",
          backgroundColor: "black",
          color: "white",
          border: "2px solid white",
          padding: "10px 20px",
          cursor: "pointer",
          transition: "transform 0.2s, background-color 0.2s",
        },
        onClick: () => navigate("/split"), // Navigate to SplitSelection
        onMouseEnter: (e) => {
          e.target.style.backgroundColor = "#FFA500";
          e.target.style.color = "black";
          e.target.style.transform = "scale(1.1)";
        },
        onMouseLeave: (e) => {
          e.target.style.backgroundColor = "black";
          e.target.style.color = "white";
          e.target.style.transform = "scale(1)";
        },
      },
      "Start"
    ),
```

