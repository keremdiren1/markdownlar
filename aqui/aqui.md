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
