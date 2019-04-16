
# Destructuring

Destructuring is an excellent way to extract data stored in objects and arrays. Is very useful to read a complete structure in only one line.
## Destructuring assignment

Is supported in node since ~v6.4.0, Chrome #49 and Firefox #41

Let's start!

## Array destructuring
In array destructuring each variable name on the array maps to the corresponding item at the same index on the destructured array.

This is our right side array.
```javascript
const bar = ["mate without sugar", "empanadas de carne", "milanesas", "asado well done"];
```

Back in time, we use a few lines to assign different items of an array to different variables
```javascript
const mate = bar[0];
const empanada = bar[1];
const milanesa = bar[2];
const asado = bar[3];
```

Now, we can do a single line assignment with destructuring
```javascript
const [mate, empanada, milanesa, asado] = bar;
```

Also, comma (`,`) is used to omit values. Thus, we can take only the necessary elements
```javascript
const [,, milanesa, asado] = bar;
console.log(milanesa); // prints "milanesas"
```

or maybe, we need to take the first element of the array and keep the rest
```javascript
const [mate, ...rest] = bar;
console.log(mate)// prints "mate without sugar"
```
---
Extra tip!  **rest operator only  works in last position**, in other cases throws syntax error
```javascript
const [...rest, asado] = foo; // syntax error
```
---

we can destructure returning values from a function call
```javascript
const f = () => [1, 2];
const [uno, dos] = f();
console.log(uno); // prints 1
```
Take only the first
```javascript
const f = () => [1, 2];
const [uno, ...rest] = f();
console.log(uno); // prints 1
```
or destrucutre a for-of loop
```javascript
for (const [index, elem] of bar.entries()) {
  // Process
}
```

Do you remember the swap function that you learn when starting to code? 
```javascript
let playerOne = ‘Matt’;
let playerTwo = ‘Alex’;
let aux;

aux = playerOne;
playerOne = playerTwo;
playerTwo = aux;
```

Other use is to swap values in one line, without using an extra variable
```javascript
let playerOne = ‘Matt’
let playerTwo = ‘Alex’

[playerOne, playerTwo] = [playerTwo, playerOne]
```
Maybe is a bit tricky, but cleaner and shorter when you get used to.

## Object destructuring
For object destructuring we use an object in the left side of an assignment. This works almost the same as Array, only with slight differences.

Our first example of object destructuring is to simplify module load. In this way, we only load the required modules.
```javascript
const { empanada, asado } = require('@some/module'); // Yes, we are using destructuring here!
```


Destructuring parameters is a good way to continue our road. 
```javascript
function f({message, number}) {
  console.log(message); // prints 'mate con empandas'
  console.log(number); // prints 20
};

const parameter = {
  number: 20,
  message: 'mate con empanadas',
};
f(parameter);
```

Also, we can assign default values for each key. Ensuring that we will never get an undefined value 
```javascript
function f({message = 'This is default', number = 0}) {
  console.log(message); // prints 'This is default'
  console.log(number); // prints 0
};

const parameter = {};
f(parameter);
```

For code simplicity, we only want one key with default value
```javascript
function f({message = 'This is default'}) {
  console.log(message); // prints 'This is default'
  console.log(number); // number is undefined
};

const parameter = {
  number: 20,
  message: 'mate con empanadas'
};

f(parameter)
```

Another use of object destructuring is to rename and map multiple similar responses to different variables names. Ex:
```javascript
const { status: statusCode0, statusText} = await request(URI_0, BODY_0, HEADERS_0);
const { status: statusCode1, statusText} = await request(URI_1, BODY_1, HEADERS_1);
```


# Conclusions
We learned what destructuring is and how it is useful in so many ways. We can use it with objects and arrays, and is an excellent idea to import modules and returning values.
Having this in mind,*and using it*, our code will be **shorter** and **cleaner** than before

Thanks for reading!
## Sources
- [javascript.info](https://javascript.info/destructuring-assignment)
- [Mozzila.org](https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Operadores/Destructuring_assignment)
- [exploringjs.com](http://exploringjs.com/es6/ch_destructuring.html)
