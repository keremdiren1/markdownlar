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

``` html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>AQUI Visual Test Environment</title>
  
  <!-- CodeMirror Dependencies -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/codemirror.min.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/codemirror.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/mode/simple.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/edit/closebrackets.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/codemirror/5.65.12/addon/comment/comment.min.js"></script>

  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    html, body {
      height: 100vh;
      overflow: hidden;
    }

    body {
      display: flex;
      flex-direction: column;
      font-family: monospace;
      background: #fff;
    }

    .header {
      height: 48px;
      display: flex;
      align-items: center;
      padding: 0 16px;
      border-bottom: 1px solid #ccc;
      background: #f8f8f8;
    }

    .logo {
      font-weight: bold;
      font-size: 18px;
    }

    .main-content {
      flex: 1;
      display: flex;
      min-height: 0;
    }

    .editor-panel {
      width: 50%;
      display: flex;
      flex-direction: column;
      border-right: 1px solid #ccc;
    }

    .CodeMirror {
      height: 100%;
      font-size: 14px;
      background: #fdf6e3;
    }

    .visualization-panel {
      width: 50%;
      position: relative;
    }

    #canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }

    .footer {
      height: 48px;
      display: flex;
      align-items: center;
      padding: 0 16px;
      border-top: 1px solid #ccc;
      background: #f8f8f8;
    }

    .button {
      padding: 8px 16px;
      margin-right: 10px;
      font-family: monospace;
      border: none;
      background: #e0e0e0;
      cursor: pointer;
    }

    .button:hover {
      background: #d0d0d0;
    }

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
    }

    .ast-panel.visible {
      display: block;
    }
  </style>
</head>
<body>
  <div class="header">
    <div class="logo">Domine, non nisi Te</div>
  </div>

  <div class="main-content">
    <div class="editor-panel">
      <textarea id="code-editor">// Aqui</textarea>
    </div>
    <div class="visualization-panel">
      <canvas id="canvas"></canvas>
    </div>
  </div>

  <div class="ast-panel" id="ast-panel">
    <pre id="ast-output"></pre>
  </div>

  <div class="footer">
    <button class="button" id="run-button">Run (Shift + Enter)</button>
    <button class="button" id="view-ast">View AST</button>
  </div>

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
