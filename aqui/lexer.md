``` javascript
// lexer.mjs
export class Token {
  constructor(type, value, line, column) {
    this.type = type;
    this.value = value;
    this.line = line;
    this.column = column;
  }
} // this part exports the Token class while also assigning a custom constructor for it.

export class Lexer {
  constructor(input) {
    this.input = input;
    this.position = 0;
    this.line = 1;
    this.column = 1;
    this.currentChar = this.input[0] || null;
  } // This part exports the Lexer class while also assigning a custom constructor for it.

  error(message) {
    throw new Error(`Lexer error at line ${this.line}, col ${this.column}: ${message}`);
  } // This is a function designed to throw errors depending on its input.

  advance() {
    this.position++;
    if (this.position >= this.input.length) {
      this.currentChar = null;
    } else {
      this.currentChar = this.input[this.position];
      this.column++;
    }
  } // This part helps the program read what's written. This part specifically helps us advance through a line.

  skipComment() {
    this.advance();
    this.advance();
    while (this.currentChar !== null && this.currentChar !== '\n') {
      this.advance();
    }
  } // This part makes it so that the program skips parts after comments.

  skipWhitespace() {
    while (this.currentChar && /\s/.test(this.currentChar)) {
      if (this.currentChar === '\n') {
        this.line++;
        this.column = 1;
      }
      this.advance();
    }
  } // This part makes it so that the program goes to the next line after it reaches to the end of the current line.

  number() {
    let result = '';
    while (this.currentChar && /\d/.test(this.currentChar)) {
      result += this.currentChar;
      this.advance();
    }
    if (this.currentChar === '.') {
      result += '.';
      this.advance();
      while (this.currentChar && /\d/.test(this.currentChar)) {
        result += this.currentChar;
        this.advance();
      }
    }
    return new Token('NUMBER', parseFloat(result), this.line, this.column);
  } // This part detects a number, puts the number to the variable named result, and then returns a Token object with ('NUMBER', parseFloat(result), this.line, this.column) as its parameters.

  identifier() {
    let result = '';
    while (this.currentChar && /[a-zA-Z0-9_.]/.test(this.currentChar)) {
      result += this.currentChar;
      this.advance();
    }
    
    // Extended keywords for conditionals
    const keywords = {
      'param': 'PARAM',
      'shape': 'SHAPE',
      'layer': 'LAYER',
      'transform': 'TRANSFORM',
      'add': 'ADD',
      'subtract': 'SUBTRACT',
      'rotate': 'ROTATE',
      'scale': 'SCALE',
      'position': 'POSITION',
      'if': 'IF',
      'else': 'ELSE',
      'endif': 'ENDIF',
      'true': 'TRUE',
      'false': 'FALSE',
      'and': 'AND',
      'or': 'OR',
      'not': 'NOT',
      'for': 'FOR',
      'from': 'FROM',
      'to': 'TO',
      'step': 'STEP',
      'union': 'UNION',
      'in': 'IN'
    };
    
    const type = keywords[result.toLowerCase()] || 'IDENTIFIER';
    return new Token(type, result, this.line, this.column);
  } // This part checks whether the given word is a keyword or not. If it's not, it returns the type as 'IDENTIFIER'.

  getNextToken() {
    while (this.currentChar !== null) {
      if (this.currentChar === '/' && this.input[this.position + 1] === '/') {
        this.skipComment();
        continue;
      } // If the current character is a comment, the program skips it.
      
      if (/\s/.test(this.currentChar)) {
        this.skipWhitespace();
        continue;
      } // If the current character is a a line end, the program goes to the next line.
      
      if (/\d/.test(this.currentChar)) {
        return this.number();
      } // If the current character is a number, the program returns it.
      
      if (/[a-zA-Z_]/.test(this.currentChar)) {
        return this.identifier();
      } // If the current character is an identifier, the program returns it.

      // Add comparison operators
      if (this.currentChar === '=' && this.input[this.position + 1] === '=') {
        this.advance();
        this.advance();
        return new Token('EQUALS', '==', this.line, this.column - 1);
      } // If the current character is 'EQUALS', the program returns a new token representing it.

      if (this.currentChar === '!' && this.input[this.position + 1] === '=') {
        this.advance();
        this.advance();
        return new Token('NOT_EQUALS', '!=', this.line, this.column - 1);
      } // If the current character is 'NOT_EQUALS', the program returns a new token representing it.

      if (this.currentChar === '<') {
        this.advance();
        if (this.currentChar === '=') {
          this.advance();
          return new Token('LESS_EQUALS', '<=', this.line, this.column - 1);
        } // If the current character is 'LESS_EQUALS', the program returns a new token representing it.
        return new Token('LESS', '<', this.line, this.column);
      } // If the current character does not have '=' in it, the program returns a new token representing 'LESS' since it had already verified the character had 'LESS'.

      if (this.currentChar === '>') {
        this.advance();
        if (this.currentChar === '=') {
          this.advance();
          return new Token('GREATER_EQUALS', '>=', this.line, this.column - 1);
        }
        return new Token('GREATER', '>', this.line, this.column);
      } // This part is the same as the one above (the part about 'LESS' and 'LESS_EQUALS'). The only difference is that this is about 'GREATER' instead of 'LESS'.

      // Basic operators and symbols
      switch (this.currentChar) {
        case '{':
          this.advance();
          return new Token('LBRACE', '{', this.line, this.column);
        case '}':
          this.advance();
          return new Token('RBRACE', '}', this.line, this.column);
        case '[':
          this.advance();
          return new Token('LBRACKET', '[', this.line, this.column);
        case ']':
          this.advance();
          return new Token('RBRACKET', ']', this.line, this.column);
        case ':':
          this.advance();
          return new Token('COLON', ':', this.line, this.column);
        case ',':
          this.advance();
          return new Token('COMMA', ',', this.line, this.column);
        case '*':
          this.advance();
          return new Token('MULTIPLY', '*', this.line, this.column);
        case '/':
          this.advance();
          return new Token('DIVIDE', '/', this.line, this.column);
        case '+':
          this.advance();
          return new Token('PLUS', '+', this.line, this.column);
        case '-':
          this.advance();
          return new Token('MINUS', '-', this.line, this.column);
        case '.':
          this.advance();
          return new Token('DOT', '.', this.line, this.column);
        case '"':
          this.advance();
          return new Token('QUOTE', '"', this.line, this.column);
        case '(':
          this.advance();
          return new Token('LPAREN', '(', this.line, this.column);
        case ')':
          this.advance();
          return new Token('RPAREN', ')', this.line, this.column);
        default:
          this.error(`Unknown character: ${this.currentChar}`);
      } // This part first finds the type of the character. It then returns a Token representing the type of the character. This part returns an error if the type of the character is not defined.
    }
    return new Token('EOF', null, this.line, this.column); // This part returns a Token which shows the fact that the file has come to an end. (EOF means End OF File)
  }
}
```
