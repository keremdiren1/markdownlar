``` module-javascript
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
