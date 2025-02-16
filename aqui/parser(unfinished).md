``` javascript
// parser.mjs
import { Token } from './lexer.mjs'; // Imports the token class.

export class Parser {
  constructor(lexer) {
    this.lexer = lexer;
    this.currentToken = this.lexer.getNextToken();
  } // This part is the constructor for the class named Parser.

  error(message) {
    throw new Error(`Parser error at line ${this.currentToken.line}, col ${this.currentToken.column}: ${message}`);
  } // This part is a function that helps with throwing errors.

  eat(tokenType) {
    if (this.currentToken.type === tokenType) {
      const token = this.currentToken;
      this.currentToken = this.lexer.getNextToken();
      return token; // This part looks whether the current Token type is the same as the inputted one or not. If it is, the program changes the current Token to the next one and returns the old one.
    } else {
      this.error(`Expected ${tokenType} but got ${this.currentToken.type}`);
    } // If the Token types do not match, an error is thrown.
  }

  // New method for parsing conditional expressions
  parseCondition() {
    let expr = this.parseLogicalOr();
    return expr;
  }

  // Parse logical OR operations
  parseLogicalOr() {
    let expr = this.parseLogicalAnd(); // Evaluates the left side of an operation like (true && false || false && true). Then, the Token changes to the next one, as specified below.

    while (this.currentToken.type === 'OR') { // Starts if the current Token is an 'OR'.
      const operator = this.currentToken.type; // Sets the operator to 'OR'.
      this.eat('OR'); // Changes to the next Token.
      expr = {
        type: 'logical_op',
        operator: 'or',
        left: expr,
        right: this.parseLogicalAnd() // Evaluates the right side of an operation. Then, the Token changes to the next one, as specified below.
      }; // Creates a 'logical_op' type expression with 'or' as its operator. The program assigns this expression to the variable named expr.
    }
    return expr;
  }

  // Parse logical AND operations
  parseLogicalAnd() {
    let expr = this.parseComparison(); // Evaluates the left side of an operation like (true && false). Then, the Token changes to the next one, as specified below.

    while (this.currentToken.type === 'AND') { // Starts if the current Token is an 'AND'.
      const operator = this.currentToken.type; // Sets the operator to 'AND'.
      this.eat('AND'); // Changes to the next Token.
      expr = {
        type: 'logical_op',
        operator: 'and',
        left: expr,
        right: this.parseComparison() // Evaluates the right side of an operation. Then, the Token changes to the next one, as specified below.
      }; // Creates a 'logical_op' type expression with 'and' as its operator. The program assigns this expression to the variable named expr.
    }
    return expr;
  }

  // Parse comparison operations
  parseComparison() {
    let expr = this.parseExpression();

    if (['EQUALS', 'NOT_EQUALS', 'LESS', 'LESS_EQUALS', 'GREATER', 'GREATER_EQUALS'].includes(this.currentToken.type)) {
      const operator = this.currentToken.type.toLowerCase();
      this.eat(this.currentToken.type);
      expr = {
        type: 'comparison',
        operator,
        left: expr,
        right: this.parseExpression()
      };
    }
    return expr;
  }

  parseExpression() {
    let node = this.parseTerm();
    while (this.currentToken.type === 'PLUS' || this.currentToken.type === 'MINUS') {
      const operator = this.currentToken.type;
      this.eat(operator);
      node = {
        type: 'binary_op',
        operator: operator.toLowerCase(),
        left: node,
        right: this.parseTerm()
      };
    }
    return node;
  }

  parseTerm() {
    let node = this.parseFactor();
    while (this.currentToken.type === 'MULTIPLY' || this.currentToken.type === 'DIVIDE') {
      const operator = this.currentToken.type;
      this.eat(operator);
      node = {
        type: 'binary_op',
        operator: operator.toLowerCase(),
        left: node,
        right: this.parseFactor()
      };
    }
    return node;
  }

  parseFactor() {
    const token = this.currentToken;
    
    if (token.type === 'NUMBER') {
      this.eat('NUMBER');
      return { type: 'number', value: token.value };
    } 
    
    if (token.type === 'TRUE' || token.type === 'FALSE') {
      this.eat(token.type);
      return { type: 'boolean', value: token.type === 'TRUE' };
    }
    
    if (token.type === 'NOT') {
      this.eat('NOT');
      return {
        type: 'unary_op',
        operator: 'not',
        operand: this.parseFactor()
      };
    }
    
    if (token.type === 'IDENTIFIER' || token.type === 'POSITION') {
      const name = token.value;
      this.eat(token.type);
      if (this.currentToken.type === 'DOT') {
        this.eat('DOT');
        const prop = this.currentToken.value;
        this.eat('IDENTIFIER');
        return { type: 'param_ref', name, property: prop };
      }
      return { type: 'identifier', name };
    }
    
    if (token.type === 'MINUS') {
      this.eat('MINUS');
      return { type: 'unary_op', operator: 'minus', operand: this.parseFactor() };
    }
    
    if (token.type === 'LBRACKET') {
      return this.parseArray();
    }
    
    if (token.type === 'QUOTE') {
      return this.parseStringLiteral();
    }
    
    if (token.type === 'LPAREN') {
      this.eat('LPAREN');
      const expr = this.parseCondition();
      this.eat('RPAREN');
      return expr;
    }
    
    this.error(`Unexpected token in factor: ${token.type}`);
  }

  // New method for parsing if statements
  parseIfStatement() {
    this.eat('IF');
    const condition = this.parseCondition();
    this.eat('LBRACE');
    
    const thenBranch = [];
    while (this.currentToken.type !== 'RBRACE' && this.currentToken.type !== 'EOF') {
      thenBranch.push(this.parseStatement());
    }
    this.eat('RBRACE');  // Make sure we consume the closing brace
    
    let elseBranch = [];
    if (this.currentToken.type === 'ELSE') {
      this.eat('ELSE');
      this.eat('LBRACE');
      while (this.currentToken.type !== 'RBRACE' && this.currentToken.type !== 'EOF') {
        elseBranch.push(this.parseStatement());
      }
      this.eat('RBRACE');
    }
    
    return {
      type: 'if_statement',
      condition,
      thenBranch,
      elseBranch
    };
  }
  parseStringLiteral() {
    let result = '';
    let t = this.lexer.getNextToken();
    while (t.type !== 'QUOTE' && t.type !== 'EOF') {
      result += t.value + ' ';
      t = this.lexer.getNextToken();
    }
    this.eat('QUOTE');
    return { type: 'string', value: result.trim() };
  }

  parseArray() {
    this.eat('LBRACKET');
    const elements = [];
    
    while (this.currentToken.type !== 'RBRACKET') {
      elements.push(this.parseExpression());
      
      // If it's not the last element, expect a comma
      if (this.currentToken.type !== 'RBRACKET') {
        this.eat('COMMA');
      }
    }
    
    this.eat('RBRACKET');
    return { type: 'array', elements };
  }
  parseParam() {
    this.eat('PARAM');
    const name = this.currentToken.value;
    this.eat('IDENTIFIER');
    const value = this.parseExpression();
    return { type: 'param', name, value };
  }

  parseShape() {
    this.eat('SHAPE');
    const shapeType = this.currentToken.value;
    this.eat('IDENTIFIER');
    let shapeName = null;
    if (this.currentToken.type === 'IDENTIFIER') {
      shapeName = this.currentToken.value;
      this.eat('IDENTIFIER');
    }
    this.eat('LBRACE');
    const params = {};
    while (this.currentToken.type !== 'RBRACE') {
      const paramName = this.currentToken.value;
      if (this.currentToken.type !== 'IDENTIFIER' && this.currentToken.type !== 'POSITION') {
        this.error(`Expected property name, got ${this.currentToken.type}`);
      }
      this.eat(this.currentToken.type);
      this.eat('COLON');
      params[paramName] = this.parseExpression();
    }
    this.eat('RBRACE');
    return { type: 'shape', shapeType, name: shapeName, params };
  }

  parseLayer() {
    this.eat('LAYER');
    const name = this.currentToken.value;
    this.eat('IDENTIFIER');
    this.eat('LBRACE');
    const commands = [];
    while (this.currentToken.type !== 'RBRACE') {
      if (this.currentToken.type === 'IF') {
        commands.push(this.parseIfStatement());
      } else if (this.currentToken.type === 'ADD') {
        this.eat('ADD');
        commands.push({ type: 'add', shape: this.currentToken.value });
        this.eat('IDENTIFIER');
      } else if (this.currentToken.type === 'SUBTRACT') {
        this.eat('SUBTRACT');
        commands.push({ type: 'subtract', shape: this.currentToken.value });
        this.eat('IDENTIFIER');
      } else if (this.currentToken.type === 'ROTATE') {
        this.eat('ROTATE');
        commands.push({ type: 'rotate', angle: this.parseExpression() });
      } else {
        this.error(`Unknown layer command: ${this.currentToken.type}`);
      }
    }
    this.eat('RBRACE');
    return { type: 'layer', name, commands };
  }

  parseTransform() {
    this.eat('TRANSFORM');
    const target = this.currentToken.value;
    this.eat('IDENTIFIER');
    this.eat('LBRACE');
    const operations = [];
    while (this.currentToken.type !== 'RBRACE') {
      if (this.currentToken.type === 'IF') {
        operations.push(this.parseIfStatement());
      } else if (this.currentToken.type === 'SCALE') {
        this.eat('SCALE');
        this.eat('COLON');
        operations.push({ type: 'scale', value: this.parseExpression() });
      } else if (this.currentToken.type === 'ROTATE') {
        this.eat('ROTATE');
        this.eat('COLON');
        operations.push({ type: 'rotate', angle: this.parseExpression() });
      } else if (this.currentToken.type === 'POSITION') {
        this.eat('POSITION');
        this.eat('COLON');
        operations.push({ type: 'translate', value: this.parseExpression() });
      } else {
        this.error(`Unknown transform command: ${this.currentToken.type}`);
      }
    }
    this.eat('RBRACE');
    return { type: 'transform', target, operations };
  }

  parseStatement() {
    let statement;
    switch (this.currentToken.type) {
      case 'IF':
        statement = this.parseIfStatement();
        break;
      case 'PARAM':
        statement = this.parseParam();
        break;
      case 'SHAPE':
        statement = this.parseShape();
        break;
      case 'LAYER':
        statement = this.parseLayer();
        break;
      case 'TRANSFORM':
        statement = this.parseTransform();
        break;
      case 'FOR':
        statement = this.parseForLoop();
        break;
      case 'ADD':
        this.eat('ADD');
        statement = { 
          type: 'add', 
          shape: this.currentToken.value 
        };
        this.eat('IDENTIFIER');
        break;
      case 'ROTATE':
        this.eat('ROTATE');
        this.eat('COLON');
        statement = {
          type: 'rotate',
          angle: this.parseExpression()
        };
        break;
      default:
        this.error(`Unexpected token: ${this.currentToken.type}`);
    }
    return statement;
  }

  parseForLoop() {
    this.eat('FOR');
    const iterator = this.currentToken.value;
    this.eat('IDENTIFIER');
    this.eat('FROM');
    const start = this.parseExpression();
    this.eat('TO');
    const end = this.parseExpression();
    
    let step = { type: 'number', value: 1 };  // Default step
    if (this.currentToken.type === 'STEP') {
      this.eat('STEP');
      step = this.parseExpression();
    }
    
    this.eat('LBRACE');
    const body = [];
    while (this.currentToken.type !== 'RBRACE' && this.currentToken.type !== 'EOF') {
      body.push(this.parseStatement());
    }
    this.eat('RBRACE');
    
    return {
      type: 'for_loop',
      iterator,
      start,
      end,
      step,
      body
    };
  }

  parseUnion() {
    this.eat('UNION');
    const name = this.currentToken.value;
    this.eat('IDENTIFIER');
    
    this.eat('LBRACE');
    const shapes = [];
    
    while (this.currentToken.type !== 'RBRACE') {
      if (this.currentToken.type === 'ADD') {
        this.eat('ADD');
        shapes.push(this.currentToken.value);
        this.eat('IDENTIFIER');
      } else {
        this.error('Expected ADD command in union block');
      }
    }
    
    this.eat('RBRACE');
    
    return {
      type: 'union',
      name,
      shapes
    };
  }

  parse() {
    const statements = [];
    while (this.currentToken.type !== 'EOF') {
      statements.push(this.parseStatement());
    }
    return statements;
  }
}
```
