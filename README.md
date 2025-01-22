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

These lines create a reference to the `<canva>` DOM element.
