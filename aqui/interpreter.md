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
  // If we're in a loop and this is a shape node, add the loop counter to the name.
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

evaluateForLoop(node) { // If the type of the node is a for loop, this part starts.
  const start = this.evaluateExpression(node.start); // Sets the start, the end, and the step of the for loop to constants.
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
  evaluateIfStatement(node) { // If the type of the node is an if statement, this part starts.
    const condition = this.evaluateExpression(node.condition); // This part creates a condition constant that has a value based on the evaluateExpression method.
    if (this.isTruthy(condition)) { // This part activates depending on the condition and the isTruthy method, which returns true or false depending on condition.
      for (const statement of node.thenBranch) {
        this.evaluateNode(statement); // Calls the evaluateNode method, which executes the code in the if statement.
      }
    } else if (node.elseBranch && node.elseBranch.length > 0) {
      for (const statement of node.elseBranch) {
        this.evaluateNode(statement); // Calls the evaluateNode method, which executes the code in the else statement (if there is one).
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
  } // This returns false only when the given value isn't 0, doesn't have 0 length, and when it's not null or undefined. It also returns a falsy value (0 etc.) if it is one.

  evaluateParam(node) {
    const value = this.evaluateExpression(node.value);
    this.env.setParameter(node.name, value);
    return value;
  } // This part creates a method that changes the value corresponding to the node's name to the value returned by evaluateExpression.

  evaluateShape(node) {
    const params = {};
    for (const [key, expr] of Object.entries(node.params)) {
      params[key] = this.evaluateExpression(expr);
    }
    return this.env.createShape(node.shapeType, node.name, params);
  } // This part creates a method that changes the parameters of a Shape with the evaluated ones (returned by evaluateExpression).

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
  } // This part creates a method that turns a Shape to a Path by taking its points. It, however, does not take its scale, position, or rotation.

  

  evaluateLayer(node) {
    const layer = {
      name: node.name,
      shapes: new Set(),  // Use Set to store shape names
      transform: {
          position: [0, 0],
          rotation: 0,
          scale: [1, 1]
      }
  }; // This part initializes a layer object.
    for (const cmd of node.commands) { // This part looks through each command in node.
      switch (cmd.type) {
        case 'add': {
            // Just store the shape name in the layer
            layer.shapes.add(cmd.shape);
            break;
        }
        case 'rotate': {
            const angle = this.evaluateExpression(cmd.angle); // It first gets the rotation angle.
            // Directly modify shapes in this layer
            for (const shapeName of layer.shapes) {
                const shape = this.env.getShape(shapeName);
                if (shape) {
                    shape.transform.rotation = (shape.transform.rotation || 0) + angle; // Updates every shape in node to make them rotate.
                }
            }
            break;
        }
      }
    }
    return layer; // The updated layer now has the needed transformations applied.
  }
  evaluateTransform(node) {
    const target = this.env.shapes.get(node.target) || this.env.layers.get(node.target); // First checks whether node.target is a shape or not. If not, it checks whether it's a layer or not.
    if (!target) { // If it's neither, the program throws an error.
      throw new Error(`Transform target not found: ${node.target}`);
    }
    for (const op of node.operations) { // The program loops through each operation in node.
      switch (op.type) {
        case 'scale': { // If it's a scale, it scales the object to the given value.
          const scaleVal = this.evaluateExpression(op.value); // This part returns the value the object should be scaled by.
          target.transform.scale = [scaleVal, scaleVal];
          break;
        }
        case 'rotate': { // If it's a rotate, it rotates the object to the given value.
          const angle = this.evaluateExpression(op.angle); // This part returns the angle the object should be rotated by.
          target.transform.rotation += angle;
          break;
        }
        case 'translate': { // If it's a translate, it translates the object by the given coordinates.
          const [x, y] = this.evaluateExpression(op.value); // This part returns the coordinates the object should be changed by.
          target.transform.position = [x, y];
          break;
        }
        default: // If the type of operation is neither of the three, the program throws an error.
          throw new Error(`Unknown transform operation: ${op.type}`);
      }
    }
    return target; // The shape/layer is returned after every operation is applied to it.
  }

  evaluateExpression(expr) {
    switch (expr.type) {
      case 'number':   return expr.value;
      case 'string':   return expr.value;
      case 'boolean':  return expr.value; // Returns the value directly if the expression is a number, string, or a boolean.
      case 'identifier': {
        if (expr.name.startsWith('param.')) {
          const paramName = expr.name.split('.')[1];
          return this.env.getParameter(paramName);
        }
        return this.env.getParameter(expr.name);
      } // If it's an identifier, the program returns the parameter associated with that identifier.
      case 'comparison': return this.evaluateComparison(expr); // Returns true or false based on the evaluation.
      case 'logical_op': return this.evaluateLogicalOp(expr); // Returns true or false based on the boolean operation.
      case 'binary_op': {
        const left = this.evaluateExpression(expr.left);
        const right = this.evaluateExpression(expr.right);
        return this.evaluateBinaryOp(expr.operator, left, right);
      } // The program returns binary operations after it evaluates them. Binary operations are operations like 1 + 2 etc.
      case 'unary_op': {
        const operand = this.evaluateExpression(expr.operand);
        if (expr.operator === 'not') return !this.isTruthy(operand);
        return expr.operator === 'minus' ? -operand : +operand;
      } // If the operation is not or minus, it returns the operand after the it uses not or minus on it.
      case 'array':
        return expr.elements.map(e => this.evaluateExpression(e)); // If the expression's an array, the program returns it after it evaluates each value in the array.
      default:
        throw new Error(`Unknown expression type: ${expr.type}`); // The program returns an error if the expression is not defined.
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
    } // This part returns the boolean value which is the result of the comparison of the two sides of a comparison. It throws an error if it does not have a comparison defined by the program.
  }

  evaluateLogicalOp(expr) {
    const left = this.evaluateExpression(expr.left); // This part evaluates the left side of the operation.
    
    // Short-circuit evaluation
    if (expr.operator === 'and') {
      return this.isTruthy(left) ? this.isTruthy(this.evaluateExpression(expr.right)) : false;
    } // This part evaluates the right side if the left side is true.
    if (expr.operator === 'or') {
      return this.isTruthy(left) ? true : this.isTruthy(this.evaluateExpression(expr.right));
    } // This part evaluates the right side if the left side is false.
    
    throw new Error(`Unknown logical operator: ${expr.operator}`); // Throws an error for logical operations not defined here.
  } // This part is basically short-circuit evaluation.

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
    } // This part basically does the binary operation and returns the result. It, however, returns an error if the operation has a division by zero or an unknown type of operation.
  }
}
```
