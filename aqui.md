# Shapes.mjs (unfinished)

``` javascript
// Base Shape class with common functionality
class Shape {
    constructor() {
        this.position = { x: 0, y: 0 };
        this.rotation = 0;
        this.scale = { x: 1, y: 1 };
    } // This part creates a constructor for the class named Shape.

    translate(dx, dy) {
        this.position.x += dx;
        this.position.y += dy;
        return this;
    } // This part creates a method that can be used to change the position of the Shape. This method also returns the Shape after it updates its position.

    rotate(angle) {
        this.rotation = (this.rotation + angle) % 360;
        return this;
    } // This part creates a method that can be used to change the rotation of the Shape. This method also returns the Shape after it updates its position.

    setScale(sx, sy) {
        this.scale.x = sx;
        this.scale.y = sy;
        return this;
    } // This part creates a method that can be used to change the scale of the Shape. This method also returns the Shape after it updates its position.

    getBoundingBox() {
        // To be implemented by subclasses
        return { x: 0, y: 0, width: 0, height: 0 };
    } // This part returns some default values for the position, width, and height of the Shape.

    // Helper method to transform points
    transformPoint(point) {
        const rad = this.rotation * Math.PI / 180;
        const cos = Math.cos(rad);
        const sin = Math.sin(rad);
        
        // Apply scale
        const scaledX = point.x * this.scale.x;
        const scaledY = point.y * this.scale.y;
        
        // Apply rotation
        const rotatedX = scaledX * cos - scaledY * sin;
        const rotatedY = scaledX * sin + scaledY * cos;
        
        // Apply translation
        return {
            x: rotatedX + this.position.x,
            y: rotatedY + this.position.y
        };
    } // This part creates a method that transforms a given point so that it conforms to the set rotation and scale.
}

// 1. Rectangle
class Rectangle extends Shape { // This part creates a subclass of the Shape class.
    constructor(width, height) {
        super();
        this.width = width;
        this.height = height;
    } // This part creates a constructor for the Rectangle class.

    getPoints() {
        const points = [
            { x: -this.width/2, y: -this.height/2 },
            { x: this.width/2, y: -this.height/2 },
            { x: this.width/2, y: this.height/2 },
            { x: -this.width/2, y: this.height/2 }
        ];
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of Rectangle.
}

// 2. Circle
class Circle extends Shape { // This part creates a subclass of of the Shape class.
    constructor(radius) {
        super();
        this.radius = radius;
    } // This part creates a constructor for the Circle class.

    getPoints(segments = 32) {
        const points = [];
        for (let i = 0; i < segments; i++) {
            const angle = (i / segments) * Math.PI * 2;
            points.push({
                x: Math.cos(angle) * this.radius,
                y: Math.sin(angle) * this.radius
            });
        }
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of Circle.
}

// 3. Triangle
class Triangle extends Shape { // This part creates a subclass of of the Shape class.
    constructor(base, height) {
        super();
        this.base = base;
        this.height = height;
    } // This part creates a constructor for the Triangle class.

    getPoints() {
        const points = [
            { x: -this.base/2, y: -this.height/2 },
            { x: this.base/2, y: -this.height/2 },
            { x: 0, y: this.height/2 }
        ];
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of Triangle.
}

// 4. Ellipse
class Ellipse extends Shape { // This part creates a subclass of of the Shape class.
    constructor(radiusX, radiusY) {
        super();
        this.radiusX = radiusX;
        this.radiusY = radiusY;
    } // This part creates a constructor for the Ellipse class.

    getPoints(segments = 32) {
        const points = [];
        for (let i = 0; i < segments; i++) {
            const angle = (i / segments) * Math.PI * 2;
            points.push({
                x: Math.cos(angle) * this.radiusX,
                y: Math.sin(angle) * this.radiusY
            });
        }
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of Ellipse.
}

// 5. Regular Polygon
class RegularPolygon extends Shape { // This part creates a subclass of of the Shape class.
    constructor(radius, sides) {
        super();
        this.radius = radius;
        this.sides = sides;
    } // This part creates a constructor for the RegularPolygon class.

    getPoints() {
        const points = [];
        for (let i = 0; i < this.sides; i++) {
            const angle = (i / this.sides) * Math.PI * 2;
            points.push({
                x: Math.cos(angle) * this.radius,
                y: Math.sin(angle) * this.radius
            });
        }
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of RegularPolygon.
}

// 6. Star
class Star extends Shape { // This part creates a subclass of of the Shape class.
    constructor(outerRadius, innerRadius, points) {
        super();
        this.outerRadius = outerRadius;
        this.innerRadius = innerRadius;
        this.points = points;
    } // This part creates a constructor for the Star class.

    getPoints() {
        const points = [];
        for (let i = 0; i < this.points * 2; i++) {
            const angle = (i / (this.points * 2)) * Math.PI * 2;
            const radius = i % 2 === 0 ? this.outerRadius : this.innerRadius;
            points.push({
                x: Math.cos(angle) * radius,
                y: Math.sin(angle) * radius
            });
        }
        return points.map(p => this.transformPoint(p));
    } // This part creates a method that returns all of the points of Star.
}

// 7. Arc
class Arc extends Shape {
    constructor(radius, startAngle, endAngle) {
        super();
        this.radius = radius;
        this.startAngle = startAngle;
        this.endAngle = endAngle;
    }

    getPoints(segments = 32) {
        const points = [];
        const angleSpan = this.endAngle - this.startAngle;
        for (let i = 0; i <= segments; i++) {
            const angle = this.startAngle + (i / segments) * angleSpan;
            const rad = angle * Math.PI / 180;
            points.push({
                x: Math.cos(rad) * this.radius,
                y: Math.sin(rad) * this.radius
            });
        }
        return points.map(p => this.transformPoint(p));
    }
}

// 8. RoundedRectangle
class RoundedRectangle extends Shape {
    constructor(width, height, radius) {
        super();
        this.width = width;
        this.height = height;
        this.radius = Math.min(radius, width/2, height/2);
    }

    getPoints(segmentsPerCorner = 8) {
        const points = [];
        const w = this.width/2;
        const h = this.height/2;
        const r = this.radius;

        // Helper function for corner arcs
        const addCorner = (centerX, centerY, startAngle) => {
            for (let i = 0; i <= segmentsPerCorner; i++) {
                const angle = startAngle + (i / segmentsPerCorner) * Math.PI/2;
                points.push({
                    x: centerX + Math.cos(angle) * r,
                    y: centerY + Math.sin(angle) * r
                });
            }
        };

        // Top right corner
        addCorner(w - r, -h + r, 0);
        // Bottom right corner
        addCorner(w - r, h - r, Math.PI/2);
        // Bottom left corner
        addCorner(-w + r, h - r, Math.PI);
        // Top left corner
        addCorner(-w + r, -h + r, -Math.PI/2);

        return points.map(p => this.transformPoint(p));
    }
}

// 9. Path
class Path extends Shape {
    constructor() {
        super();
        this.points = [];
        this.closed = false;
    }

    addPoint(x, y) {
        this.points.push({ x, y });
        return this;
    }

    close() {
        this.closed = true;
        return this;
    }

    getPoints() {
        return this.points.map(p => this.transformPoint(p));
    }
}

// 10. Arrow
class Arrow extends Shape {
    constructor(length, headWidth, headLength) {
        super();
        this.length = length;
        this.headWidth = headWidth;
        this.headLength = headLength;
    }

    getPoints() {
        const points = [
            { x: 0, y: 0 },
            { x: this.length - this.headLength, y: 0 },
            { x: this.length - this.headLength, y: -this.headWidth/2 },
            { x: this.length, y: 0 },
            { x: this.length - this.headLength, y: this.headWidth/2 },
            { x: this.length - this.headLength, y: 0 }
        ];
        return points.map(p => this.transformPoint(p));
    }
}

// 11. Text (Bounding box for text)
class Text extends Shape {
    constructor(text, fontSize = 12, fontFamily = 'Arial') {
        super();
        this.text = text;
        this.fontSize = fontSize;
        this.fontFamily = fontFamily;
        // Rough estimation of text dimensions
        this.width = this.fontSize * 0.6 * this.text.length;
        this.height = this.fontSize;
    }

    getBoundingBox() {
        return {
            x: this.position.x,
            y: this.position.y,
            width: this.width,
            height: this.height
        };
    }
}

// 12. Bezier Curve
class BezierCurve extends Shape {
    constructor(startPoint, controlPoint1, controlPoint2, endPoint) {
        super();
        this.startPoint = startPoint;
        this.controlPoint1 = controlPoint1;
        this.controlPoint2 = controlPoint2;
        this.endPoint = endPoint;
    }

    getPoints(segments = 32) {
        const points = [];
        for (let t = 0; t <= 1; t += 1/segments) {
            const point = this.calculateBezierPoint(t);
            points.push(point);
        }
        return points.map(p => this.transformPoint(p));
    }

    calculateBezierPoint(t) {
        const x = Math.pow(1-t, 3) * this.startPoint.x +
                 3 * Math.pow(1-t, 2) * t * this.controlPoint1.x +
                 3 * (1-t) * Math.pow(t, 2) * this.controlPoint2.x +
                 Math.pow(t, 3) * this.endPoint.x;
        const y = Math.pow(1-t, 3) * this.startPoint.y +
                 3 * Math.pow(1-t, 2) * t * this.controlPoint1.y +
                 3 * (1-t) * Math.pow(t, 2) * this.controlPoint2.y +
                 Math.pow(t, 3) * this.endPoint.y;
        return { x, y };
    }
}

// 13. Donut (Annulus)
class Donut extends Shape {
    constructor(outerRadius, innerRadius) {
        super();
        this.outerRadius = outerRadius;
        this.innerRadius = innerRadius;
    }

    getPoints(segments = 32) {
        const points = [];
        // Outer circle
        for (let i = 0; i < segments; i++) {
            const angle = (i / segments) * Math.PI * 2;
            points.push({
                x: Math.cos(angle) * this.outerRadius,
                y: Math.sin(angle) * this.outerRadius
            });
        }
        // Inner circle (in reverse to create hole)
        for (let i = segments - 1; i >= 0; i--) {
            const angle = (i / segments) * Math.PI * 2;
            points.push({
                x: Math.cos(angle) * this.innerRadius,
                y: Math.sin(angle) * this.innerRadius
            });
        }
        return points.map(p => this.transformPoint(p));
    }
}

// 14. Spiral
class Spiral extends Shape {
    constructor(startRadius, endRadius, turns) {
        super();
        this.startRadius = startRadius;
        this.endRadius = endRadius;
        this.turns = turns;
    }

    getPoints(segments = 100) {
        const points = [];
        const totalAngle = this.turns * Math.PI * 2;
        for (let i = 0; i <= segments; i++) {
            const t = i / segments;
            const angle = t * totalAngle;
            const radius = this.startRadius + (this.endRadius - this.startRadius) * t;
            points.push({
                x: Math.cos(angle) * radius,
                y: Math.sin(angle) * radius
            });
        }
        return points.map(p => this.transformPoint(p));
    }
}

// 15. Cross
class Cross extends Shape {
    constructor(width, thickness) {
        super();
        this.width = width;
        this.thickness = thickness;
    }

    getPoints() {
        const w = this.width/2;
        const t = this.thickness/2;
        const points = [
            { x: -t, y: -w }, { x: t, y: -w },
            { x: t, y: -t }, { x: w, y: -t },
            { x: w, y: t }, { x: t, y: t },
            { x: t, y: w }, { x: -t, y: w },
            { x: -t, y: t }, { x: -w, y: t },
            { x: -w, y: -t }, { x: -t, y: -t }
        ];
        return points.map(p => this.transformPoint(p));
    }
}

// 16. Gear
class Gear extends Shape {
    constructor(pitch_diameter, teeth, pressure_angle = 20) {
        super();
        this.pitch_diameter = pitch_diameter;
        this.teeth = teeth;
        this.pressure_angle = pressure_angle * Math.PI / 180;
        this.addendum = this.pitch_diameter / this.teeth;
        this.dedendum = 1.25 * this.addendum;
    }

    getPoints(points_per_tooth = 4) {
        const points = [];
        const pitch_point = this.pitch_diameter / 2;
        const base_radius = pitch_point * Math.cos(this.pressure_angle);
        const outer_radius = pitch_point + this.addendum;
        const root_radius = pitch_point - this.dedendum;

        for (let i = 0; i < this.teeth; i++) {
            const angle = (i / this.teeth) * Math.PI * 2;
            for (let j = 0; j < points_per_tooth; j++) {
                const t = j / points_per_tooth;
                const tooth_angle = (Math.PI * 2) / (this.teeth * 2);
                const current_angle = angle + tooth_angle * t;
                
                // Generate tooth profile
                const radius = t < 0.5 ? outer_radius : root_radius;
                points.push({
                    x: Math.cos(current_angle) * radius,
                    y: Math.sin(current_angle) * radius
                });
            }
        }
        return points.map(p => this.transformPoint(p));
    }
}

// 17. Wave
class Wave extends Shape {
    constructor(width, amplitude, frequency) {
        super();
        this.width = width;
        this.amplitude = amplitude;
        this.frequency = frequency;
    }

    getPoints(segments = 50) {
        const points = [];
        for (let i = 0; i <= segments; i++) {
            const x = (i / segments) * this.width - this.width/2;
            const y = Math.sin((x * this.frequency * Math.PI * 2) / this.width) * this.amplitude;
            points.push({ x, y });
        }
        return points.map(p => this.transformPoint(p));
    }
}

// 18. Slot
class Slot extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
        this.radius = width/2;
    }

    getPoints(segments = 32) {
        const points = [];
        const centerDist = (this.length - this.width)/2;

        // Add first semicircle
        for (let i = 0; i <= segments/2; i++) {
            const angle = Math.PI + (i / (segments/2)) * Math.PI;
            points.push({
                x: -centerDist + Math.cos(angle) * this.radius,
                y: Math.sin(angle) * this.radius
            });
        }

        // Add second semicircle
        for (let i = 0; i <= segments/2; i++) {
            const angle = (i / (segments/2)) * Math.PI;
            points.push({
                x: centerDist + Math.cos(angle) * this.radius,
                y: Math.sin(angle) * this.radius
            });
        }

        return points.map(p => this.transformPoint(p));
    }
}

// 19. Chamfer Rectangle
class ChamferRectangle extends Shape {
    constructor(width, height, chamfer) {
        super();
        this.width = width;
        this.height = height;
        this.chamfer = Math.min(chamfer, width/2, height/2);
    }

    getPoints() {
        const w = this.width/2;
        const h = this.height/2;
        const c = this.chamfer;
        
        const points = [
            { x: -w + c, y: -h },
            { x: w - c, y: -h },
            { x: w, y: -h + c },
            { x: w, y: h - c },
            { x: w - c, y: h },
            { x: -w + c, y: h },
            { x: -w, y: h - c },
            { x: -w, y: -h + c }
        ];
        
        return points.map(p => this.transformPoint(p));
    }
}

// 20. Polygon with Holes
class PolygonWithHoles extends Shape {
    constructor(outerPoints, holes = []) {
        super();
        this.outerPoints = outerPoints;
        this.holes = holes; // Array of arrays of points
    }

    getPoints() {
        let allPoints = [...this.outerPoints];
        
        // Add holes in reverse order to create proper path
        this.holes.forEach(hole => {
            allPoints.push(allPoints[0]); // Bridge to hole
            allPoints.push(...hole.reverse()); // Add hole points
            allPoints.push(hole[0]); // Close hole
        });
        
        return allPoints.map(p => this.transformPoint(p));
    }
}

// Utility functions for shape operations
const ShapeUtils = {
    // Boolean operations
    union(shape1, shape2) {
        // Implementation would require complex polygon clipping library
        throw new Error("Boolean operations require additional geometry library");
    },

    // Distance between shapes
    distance(shape1, shape2) {
        const bb1 = shape1.getBoundingBox();
        const bb2 = shape2.getBoundingBox();
        
        return Math.sqrt(
            Math.pow(bb1.x - bb2.x, 2) + 
            Math.pow(bb1.y - bb2.y, 2)
        );
    },

    // Check if point is inside shape
    pointInShape(point, shape) {
        const points = shape.getPoints();
        let inside = false;
        
        for (let i = 0, j = points.length - 1; i < points.length; j = i++) {
            const xi = points[i].x, yi = points[i].y;
            const xj = points[j].x, yj = points[j].y;
            
            const intersect = ((yi > point.y) !== (yj > point.y))
                && (point.x < (xj - xi) * (point.y - yi) / (yj - yi) + xi);
            
            if (intersect) inside = !inside;
        }
        
        return inside;
    },

    // Convert shape to SVG path
    toSVGPath(shape) {
        const points = shape.getPoints();
        if (points.length === 0) return '';
        
        let path = `M ${points[0].x} ${points[0].y}`;
        for (let i = 1; i < points.length; i++) {
            path += ` L ${points[i].x} ${points[i].y}`;
        }
        path += ' Z';
        
        return path;
    }
};

export {
    Shape,
    Rectangle,
    Circle,
    Triangle,
    Ellipse,
    RegularPolygon,
    Star,
    Arc,
    RoundedRectangle,
    Path,
    Arrow,
    Text,
    BezierCurve,
    Donut,
    Spiral,
    Cross,
    Gear,
    Wave,
    Slot,
    ChamferRectangle,
    PolygonWithHoles,
    ShapeUtils
};
```

# environment.mjs

``` javascript
// environment.mjs
export class Environment {
    constructor() {
      this.parameters = new Map();
      this.shapes = new Map();
      this.layers = new Map();
    } // This part creates a constructor for the Environment class.
  
    getParameter(name) {
      if (!this.parameters.has(name)) {
        throw new Error(`Parameter not found: ${name}`);
      }
      return this.parameters.get(name);
    } // This part creates a method that gives an error if the given name is not found. If it is found, it returns the value corresponding to the name.
  
    setParameter(name, value) {
      this.parameters.set(name, value);
    } // This part creates a method that adds a new name and a value corresponding to it. If the name already exists, it replaces the value corresponding to that name with the new one.
  
    createShape(type, name, params) {
      const shape = {
        type,
        id: `${type}_${name}_${Date.now()}`,
        params: { ...params },
        transform: {
          position: params.position || [0, 0],
          rotation: 0,
          scale: [1, 1]
        }
      };
      this.shapes.set(name, shape);
      return shape;
    } // This part creates a method that creates a shape constant depending on the inputs. Its type, name, id, and position is determined by what was inputted in the parameters in this function. After creation, this shape is added to the Map object named shapes and set to correspond to the name inputted in the methods parameters.
  
    getShape(name) {
      if (!this.shapes.has(name)) {
        throw new Error(`Shape not found: ${name}`);
      }
      return this.shapes.get(name);
    } // This part creates a method that gives an error if the given name is not found. If it is found, it returns the shape corresponding to that name.
  
    createLayer(name) {
      const layer = {
        name,
        shapes: new Map(),
        operations: [],
        transform: {
          position: [0, 0],
          rotation: 0,
          scale: [1, 1]
        }
      };
      this.layers.set(name, layer);
      return layer;
    } // This part creates a method that creates a layer constant that sets the layer to layers Map object with the given layer's corresponding name. This method also returns the newly created layer in the last line.
  }
```

# index.html (unfinished)

``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>AQUI Visual Test Environment</title> <!-- This part puts a title on the screen. -->
  
  <!-- CodeMirror Dependencies -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/codemirror.min.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/codemirror.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/mode/simple.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/edit/closebrackets.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/comment/comment.min.js"></script> <!-- This part imports needed stuff for the code. -->

  <style> <!-- this part creates a style to use in the code. -->
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    } <!-- This part is creates a use for * which resets the padding and the margin. -->

    html, body {
      height: 100vh;
      overflow: hidden;
    } <!-- This part makes the <html> and <body> types have heights the goes from the bottom to the top of the screen. -->

    body {
      display: flex;
      flex-direction: column;
      font-family: monospace;
      background: #fff;
    } <!-- This part makes the <body> type have a font of monospace and a white background. It also makes the display type of <body> to flex. -->

    .header {
      height: 48px;
      display: flex;
      align-items: center;
      padding: 0 16px;
      border-bottom: 1px solid #ccc;
      background: #f8f8f8;
    } <!-- This part makes the class="header" to have a 48px height, have a 0 to 16px padding, have a light gray border at the bottom, and have a light gray background that is really white. -->

    .logo {
      font-weight: bold;
      font-size: 18px;
    } <!-- This part makes the class="logo" to have a bold text which has 18px size. -->

    .main-content {
      flex: 1;
      display: flex;
      min-height: 0;
    } <!-- This part makes the class="main-content" to have a display type of flex and a minimum height of 0. -->

    .editor-panel {
      width: 50%;
      display: flex;
      flex-direction: column;
      border-right: 1px solid #ccc;
    } <!-- This part makes the class="editor-panel" to have a width of 50% (50% of its parent element), a display type of flex, a flex direction of column, and a light gray border to the right. -->

    .CodeMirror {
      height: 100%;
      font-size: 14px;
      background: #fdf6e3;
    } <!-- This part makes the class="CodeMirror" to have a 100% height (100% of its parent element), a 14px font, and a very light, warm beige color background. -->

    .visualization-panel {
      width: 50%;
      position: relative;
    } <!-- This part makes the class="visualization-panel" have a 50% width and a position type of relative (which is set relative to an element with position: absolute). -->

    #canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    } <!-- This part makes the elements with the canvas id have a position type of absolute, a position at the top left, and a width and height of 100%. -->

    .footer {
      height: 48px;
      display: flex;
      align-items: center;
      padding: 0 16px;
      border-top: 1px solid #ccc;
      background: #f8f8f8;
    } <!-- This part makes the class="footer" have a height of 48px, have a display type of flex, have its items aligned in the middle, have a padding of 0 to 16px, have a light gray border on top, and have a very light gray background. -->

    .button {
      padding: 8px 16px;
      margin-right: 10px;
      font-family: monospace;
      border: none;
      background: #e0e0e0;
      cursor: pointer;
    } <!-- This part makes the class="button" have a padding of 8px to 16px, have a 10px margin from the element to its right, have a font of monospace, have a light gray background, and change the cursor to a pointer when it hovers on it. -->

    .button:hover {
      background: #d0d0d0;
    } <!-- This part makes it so that the background turns into a medium light gray color when the mouse hovers on it. -->

    .ast-panel {
      position: absolute;
      bottom: 48px;
      left: 0;
      right: 0;
      height: 200px;
      background: white;
      border-top: 1px solid #ccc;
      padding: 16px;
      display: none;
      font-family: monospace;
      font-size: 12px;
      overflow: auto;
    } <!-- This part makes the class="ast-panel" have a position type of absolute, have a position 48px above the bottom, have it stretch from the leftmost part to the rightmost part of its parent element, have a height of 200px, have a white background, have a light gray border on its top side, have a padding of 16px, have a monospace text font, and have a 12px text size. This element is invisible by default. -->

    .ast-panel.visible {
      display: block;
    } <!-- This part is used to make the class="ast-panel" visible. -->
  </style>
</head>
<body>
  <div class="header">
    <div class="logo">Domine, non nisi Te</div>
  </div> <!-- This part creates a header that has "Domine, non tisi Te" written in it. -->

  <div class="main-content">
    <div class="editor-panel">
      <textarea id="code-editor">// Aqui</textarea>
    </div>
    <div class="visualization-panel">
      <canvas id="canvas"></canvas>
    </div>
  </div> <!-- This part creates a panel we can write in. This has "// Aqui" written before we start to write. This part also sets an id to this panel. -->

  <div class="ast-panel" id="ast-panel">
    <pre id="ast-output"></pre>
  </div> <!--  -->

  <div class="footer">
    <button class="button" id="run-button">Run (Shift + Enter)</button>
    <button class="button" id="view-ast">View AST</button>
  </div> <!-- This part creates a footer that has 2 buttons in it. The first button has an id of "run-button" and has "Run (Shift + Enter)" written in it. The second button has an id of "view-ast" and has "View AST" written in it. -->

  <script>
    // Define Aqui syntax for CodeMirror
    CodeMirror.defineSimpleMode("aqui", {
      start: [
        { regex: /\/\/.*/, token: "comment" },    // Comments
        { regex: /\b(param|shape|layer|transform|add|subtract|rotate|scale|position)\b/, token: "keyword" },
        { regex: /\b\d+(\.\d+)?\b/, token: "number" },  // Numbers
        { regex: /"(?:[^\\]|\\.)*?"/, token: "string" },  // Strings
        { regex: /[\{\}\[\]:,]/, token: "operator" }  // Operators
      ]
    });

    // Initialize CodeMirror
    const editor = CodeMirror.fromTextArea(document.getElementById('code-editor'), {
      mode: "aqui",
      theme: "default",
      lineNumbers: true,
      autoCloseBrackets: true,
      matchBrackets: true
    });

    // Preload example code
    editor.setValue(`//Aqui`);

    (async () => {
      try {
        const { Lexer } = await import('./lexer.mjs');
        const { Parser } = await import('./parser.mjs');
        const { Interpreter } = await import('./interpreter.mjs');
        const { Renderer } = await import('./renderer.mjs');

        const canvas = document.getElementById('canvas');
        const renderer = new Renderer(canvas);
        const runButton = document.getElementById('run-button');
        const viewAstButton = document.getElementById('view-ast');
        const astPanel = document.getElementById('ast-panel');
        const astOutput = document.getElementById('ast-output');

        function runCode() {
          try {
            renderer.clear();
            const code = editor.getValue();
            const lexer = new Lexer(code);
            const parser = new Parser(lexer);
            const ast = parser.parse();
            astOutput.textContent = JSON.stringify(ast, null, 2);

            const interpreter = new Interpreter();
            const result = interpreter.interpret(ast);

            for (const shape of result.shapes.values()) {
              renderer.drawShape(shape);
            }

            for (const layer of result.layers.values()) {
              const { transform } = layer;
              for (const shape of layer.shapes.values()) {
                const combinedShape = {
                  ...shape,
                  transform: {
                    position: shape.transform.position,
                    rotation: (shape.transform.rotation || 0) + (transform.rotation || 0),
                    scale: [
                      shape.transform.scale[0] * transform.scale[0],
                      shape.transform.scale[1] * transform.scale[1]
                    ]
                  }
                };
                renderer.drawShape(combinedShape);
              }
            }
          } catch (error) {
            console.error(error);
          }
        }

        runButton.addEventListener('click', runCode);
        viewAstButton.addEventListener('click', () => {
          astPanel.classList.toggle('visible');
        });

        editor.on("keydown", (cm, event) => {
          if (event.shiftKey && event.key === "Enter") {
            event.preventDefault();
            runCode();
          }
        });

        window.addEventListener("resize", () => {
          renderer.setupCanvas();
          runCode();
        });

      } catch (error) {
        console.error('Module import error:', error);
      }
    })();
  </script>
</body>
</html>
```

# interpreter.mjs (unfinished)

``` javascript
// interpreter.mjs
import { Environment } from './environment.mjs'; // This line imports the Environment class from environment.mjs

export class Interpreter {
  constructor() {
    this.env = new Environment();
  } // This part creates a constructor for the Interpreter class.

  interpret(ast) {
    let result = null;
    for (const node of ast) {
      result = this.evaluateNode(node); // This part calls the evaluateNode method and replaces the value of result with the returned value. The evaluateNode method will be explained below.
    }
    return {
      parameters: this.env.parameters,
      shapes: this.env.shapes,
      layers: this.env.layers,
      result
    }; // This part returns the parameters, shapes, and layers in env. This part also returns the variable named result.
  }

 // Add to Interpreter class
evaluateNode(node) {
  // If we're in a loop and this is a shape node, add the loop counter to the name
  if (node.type === 'shape' && this.currentLoopCounter !== undefined) {
    node = {
      ...node,
      name: `${node.name}_${this.currentLoopCounter}`
    };
  }

  switch (node.type) { // When this line activates, a specific case starts depending on the type of the node.
    case 'param': 
      return this.evaluateParam(node);
    
    case 'shape':
      return this.evaluateShape(node);
    
    case 'layer':
      return this.evaluateLayer(node);
    
    case 'transform':
      return this.evaluateTransform(node);
    
    case 'if_statement':
      return this.evaluateIfStatement(node);
    
    case 'for_loop':
      return this.evaluateForLoop(node);
    
    default:
      throw new Error(`Unknown node type: ${node.type}`);
  } // Each type has its own evaluate method called. If it is not a type defined here, the program throws an error.
}

evaluateForLoop(node) {
  const start = this.evaluateExpression(node.start);
  const end = this.evaluateExpression(node.end);
  const step = this.evaluateExpression(node.step);
  
  // Save current loop state if we're in nested loops
  const outerLoopCounter = this.currentLoopCounter;
  
  // Execute the loop
  for (let i = start; i <= end; i += step) {
    // Set the iterator value in the environment
    this.env.setParameter(node.iterator, i);
    
    // Update the current loop counter
    this.currentLoopCounter = i;
    
    // Execute each statement in the loop body
    for (const statement of node.body) {
      this.evaluateNode(statement);
    }
  }
  
  // Restore previous loop state
  this.currentLoopCounter = outerLoopCounter;
  
  // Clean up the iterator
  this.env.parameters.delete(node.iterator);
}
  evaluateIfStatement(node) {
    const condition = this.evaluateExpression(node.condition);
    if (this.isTruthy(condition)) {
      for (const statement of node.thenBranch) {
        this.evaluateNode(statement);
      }
    } else if (node.elseBranch && node.elseBranch.length > 0) {
      for (const statement of node.elseBranch) {
        this.evaluateNode(statement);
      }
    }
  }

  isTruthy(value) {
    if (typeof value === 'boolean') return value;
    if (typeof value === 'number') return value !== 0;
    if (typeof value === 'string') return value.length > 0;
    if (Array.isArray(value)) return value.length > 0;
    if (value === null || value === undefined) return false;
    return true;
  }

  evaluateParam(node) {
    const value = this.evaluateExpression(node.value);
    this.env.setParameter(node.name, value);
    return value;
  }

  evaluateShape(node) {
    const params = {};
    for (const [key, expr] of Object.entries(node.params)) {
      params[key] = this.evaluateExpression(expr);
    }
    return this.env.createShape(node.shapeType, node.name, params);
  }

  shapeToPath(shape) {
    // Convert any shape to a path using ShapePoints
    const points = ShapePoints.getPoints(shape);
    return {
      type: 'path',
      params: {
        points,
        closed: true
      },
      transform: {
        position: [0, 0],
        rotation: 0,
        scale: [1, 1]
      }
    };
  }

  

  evaluateLayer(node) {
    const layer = {
      name: node.name,
      shapes: new Set(),  // Use Set to store shape names
      transform: {
          position: [0, 0],
          rotation: 0,
          scale: [1, 1]
      }
  };
    for (const cmd of node.commands) {
      switch (cmd.type) {
        case 'add': {
            // Just store the shape name in the layer
            layer.shapes.add(cmd.shape);
            break;
        }
        case 'rotate': {
            const angle = this.evaluateExpression(cmd.angle);
            // Directly modify shapes in this layer
            for (const shapeName of layer.shapes) {
                const shape = this.env.getShape(shapeName);
                if (shape) {
                    shape.transform.rotation = (shape.transform.rotation || 0) + angle;
                }
            }
            break;
        }
      }
    }
    return layer;
  }
  evaluateTransform(node) {
    const target = this.env.shapes.get(node.target) || this.env.layers.get(node.target);
    if (!target) {
      throw new Error(`Transform target not found: ${node.target}`);
    }
    for (const op of node.operations) {
      switch (op.type) {
        case 'scale': {
          const scaleVal = this.evaluateExpression(op.value);
          target.transform.scale = [scaleVal, scaleVal];
          break;
        }
        case 'rotate': {
          const angle = this.evaluateExpression(op.angle);
          target.transform.rotation += angle;
          break;
        }
        case 'translate': {
          const [x, y] = this.evaluateExpression(op.value);
          target.transform.position = [x, y];
          break;
        }
        default:
          throw new Error(`Unknown transform operation: ${op.type}`);
      }
    }
    return target;
  }

  evaluateExpression(expr) {
    switch (expr.type) {
      case 'number':   return expr.value;
      case 'string':   return expr.value;
      case 'boolean':  return expr.value;
      case 'identifier': {
        if (expr.name.startsWith('param.')) {
          const paramName = expr.name.split('.')[1];
          return this.env.getParameter(paramName);
        }
        return this.env.getParameter(expr.name);
      }
      case 'comparison': return this.evaluateComparison(expr);
      case 'logical_op': return this.evaluateLogicalOp(expr);
      case 'binary_op': {
        const left = this.evaluateExpression(expr.left);
        const right = this.evaluateExpression(expr.right);
        return this.evaluateBinaryOp(expr.operator, left, right);
      }
      case 'unary_op': {
        const operand = this.evaluateExpression(expr.operand);
        if (expr.operator === 'not') return !this.isTruthy(operand);
        return expr.operator === 'minus' ? -operand : +operand;
      }
      case 'array':
        return expr.elements.map(e => this.evaluateExpression(e));
      default:
        throw new Error(`Unknown expression type: ${expr.type}`);
    }
  }

  evaluateComparison(expr) {
    const left = this.evaluateExpression(expr.left);
    const right = this.evaluateExpression(expr.right);
    
    switch (expr.operator) {
      case 'equals':         return left === right;
      case 'not_equals':     return left !== right;
      case 'less':          return left < right;
      case 'less_equals':    return left <= right;
      case 'greater':       return left > right;
      case 'greater_equals': return left >= right;
      default:
        throw new Error(`Unknown comparison operator: ${expr.operator}`);
    }
  }

  evaluateLogicalOp(expr) {
    const left = this.evaluateExpression(expr.left);
    
    // Short-circuit evaluation
    if (expr.operator === 'and') {
      return this.isTruthy(left) ? this.isTruthy(this.evaluateExpression(expr.right)) : false;
    }
    if (expr.operator === 'or') {
      return this.isTruthy(left) ? true : this.isTruthy(this.evaluateExpression(expr.right));
    }
    
    throw new Error(`Unknown logical operator: ${expr.operator}`);
  }

  evaluateBinaryOp(operator, left, right) {
    switch (operator) {
      case 'plus':     return left + right;
      case 'minus':    return left - right;
      case 'multiply': return left * right;
      case 'divide':
        if (right === 0) throw new Error('Division by zero');
        return left / right;
      default:
        throw new Error(`Unknown binary operator: ${operator}`);
    }
  }
}
```

# lexer.mjs (unfinished)

``` javascript
```

# parser.mjs (unfinished)

``` javascript
```

# renderer.mjs (unfinished)

``` javascript
```
