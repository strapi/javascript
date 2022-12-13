# Strapi Javascript Style Guide

## Table of Contents

1. [Classes](#classes)
2. [Modules & Constructors](#modules-and-constructors)

## Classes

**Airbnb ref**: https://github.com/airbnb/javascript#classes--constructors

### Prefer object instead of class instance for modules

**Bad:**

```js
class SomeClass {
  foo() {}
}

module.exports = new SomeClass();
```

**Good:**

```js
const myModule = {
  foo() {};
}

module.exports = myModule
```

**[⬆ back to top](#table-of-contents)**

---

### Prefer exporting factories over Classes

Exporting classes forces the caller to use `new X()` This makes it harder to change the internal implementation without breaking the callers.
Using factories allows us to change the implementation without changing the callers.
You can read more in this [article](https://medium.com/javascript-scene/master-the-javascript-interview-what-s-the-difference-between-class-prototypal-inheritance-e4cd0a7562e9).

**Bad:**

```js
class SomeClass {}

module.exports = SomeClass;
```

**Good:**

```js
class SomeClass {}

module.exports = (...args) => {
  return new SomeClass(...args);
};
```

**[⬆ back to top](#table-of-contents)**

---

## Modules and Constructors

**[⬆ back to top](#table-of-contents)**

### A module is never instantiated. 

It is an interface to functions you can use independently or not from the others

**Option 1**:

```js
/*
 * Good: module.exports is at the top of the file
 * Bad: Using function declaration + relying on hoisting
 */

module.exports = {
	foo,
	bar
};

function foo() {}
function bar() {}
```

**Option 2**:

```js
/*
 * Good: 
 *   - Using function expressions
 *   - Easy to call each function from the others
 *   - 
 * Bad: 
 *   - Hard to differentiate between private and public methods 
 *   - Less readable
 *   - Can be bound but won't be clear what this is
 */

const foo = function foo() {}
const bar = function bar() {} 

module.exports = {
	foo,
	bar
};
```

**Option 3**:

```js
/*
 * Good: 
 *   - Using function expressions
 *   - Easy to call each function from the others
 *   - Brevity
 *   - Prevent bad this bindings
 * Bad: 
 *   - Hard to differentiate between private and public methods 
 *   - Can feel a bit too spaghetti with many functions
 *   - Prevent this bindings
 *   - Not usable as object methods (no this possible => no state)
 */

const privateMethod = () => {};
const foo = () => {}
const bar = () => {} 

module.exports = {
	foo,
	bar
};
```

**Option 4**:

```js
/*
 * Good: 
 *   - Basically the same as option 4 but can use `interface` to reference other function instead of this
 *   - Object pattern with internal state (can use this)
 *   - Method shortand is nice to read
 *   - Easy to differentiate private / public methods
 * Bad: 
 *   - Methods can be bound incorrectly if dereferenced (invalid this / or no this)
 *   - Uses either this or module is not really good
 *   - Allows composition 🔥 with this bindings
 */

const privateMethod = () => {};

const module = {
  foo() {
    module.bar() 
  },
  bar() {}
}

module.exports = module;
```

**Option 5**:

```js
/*
 * Good: 
 *   - Basically the same as option 4 but can use `interface` to reference other function instead of this
 *   - Object pattern with internal state (can use this)
 *   - Method shortand is nice to read
 *   - Easy to differentiate private / public methods
 * Bad: 
 *   - Methods can be bound incorrectly if dereferenced (invalid this / or no this)
 *   - Uses either this or module is not really good
 *   - Allows composition 🔥 with this bindings
 */

const privateMethod = () => {};

const module = {
  foo() {
    module.bar() 
  },
  bar() {}
}

module.exports = module;
```

**Option 6**:

```js
/*
 * Good: 
 *   - Easy to diff what is internal vs external (private vs public)
 * Bad: 
 *   - No central place to know the complete interface of the module (vs a single module.exports)
 *   - Verbose
 */

const privateMethod = () => {};

exports.publicMethod = () => { privateMethod() };
exports.otherPubMethod = () =>  { exports.publicMethod() };

// or similar
const internals = {};

internals.privateMethod = () => {};

exports.publicMethod = () => { internals.privateMethod() };
exports.otherPubMethod = () =>  { exports.publicMethod() };
```

- Option 1 & 2 can be ruled out right now.
- Option 3 & (4/5) are really good. I like the structure of 4/5 and the brevity of 3.
- Option 6 is really easy to reason about when coding but more verbose.

**[⬆ back to top](#table-of-contents)**

---

### Constructor

Here you want to pass parameters to the object before instantiation

**Option 1**:

```js
class SomeClass {
  constructor(options) {
    this.x = options.x;
  }

  // when available
  #privateMethod() {}

  publicMethod() {
    this.#privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }
}


module.exports = SomeClass;
```

**Option 2**:

```js
class SomeClass {
  constructor(options) {
    this.x = options.x;
  }

  // when available
  #privateMethod() {}

  publicMethod() {
    this.#privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }
}

module.exports = (options) => new SomeClass(options);

//

const createSomeclass = require('some-class');

createSomeclass(options);
```

**Option 3**:

```js
class Controller {
  constructor(options) {
    this.x = options.x;
  }
 
  // when available
  #privateMethod() {}

  publicMethod() {
    this.#privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }

  // use this to instantiate
  static create(opts) {
	  return new Controller(opts);
  }
}

module.exports = Controller;

//
const Controller = require('controller');

Controller.create();
```

**Option 4**:

```js
function SomeClass(options) {
  this.x = options.x;

	this.publicMethodThatNeedsPrivateOptions = () => {
    console.log(option.privateKey);
  }
}

const privateMethod = () => {};

SomeClass.prototype = {
  publicMethod() {
    privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }
}

module.exports = (options) => new SomeClass(options);
```

**Option 5**:

```js
const privateMethod = () => {};

const prototype = {
  init(options) {
		this.x = options.x;

		this.publicMethodThatNeedsPrivateOptions = () => {
      console.log(option.privateKey);
    }
  },

	publicMethod() {
    privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }
}

module.exports = (options) => {
  const instance = Object.create(prototype);
  return instance.init(options)
}
```

**Option 6**:

```js
const privateMethod = () => {};

const prototype = {
	publicMethod() {
    privateMethod();
  }

  otherPubMethod() {
    this.publicMethod();
  }
}

module.exports = (options) => {
  const instance = Object.create(prototype);
  
   instance.x = options.x;
	 instance.publicMethodThatNeedsPrivateOptions = () => {
      console.log(option.privateKey);
   };

   return instance;
};
```

**Option 7**:

```js
const factory = (options) => {

  const { x } = options;

	const privateMethod = () => {};
  const publicMethod = () => { privateMethod() };
  const otherPubMethod = () => { publicMethod() };
	const publicMethodThatNeedsPrivateOptions = () => {
    console.log(option.privateKey);
  };

  return {
    publicMethod,
    otherPubMethod,
    publicMethodThatNeedsPrivateOptions,
  };
};

module.exports = factory
```

**[⬆ back to top](#table-of-contents)**

---