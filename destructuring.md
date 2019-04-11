# Destructuring

## Destructuring assignment

Destructuring is a n excellent way to extract data stored in objects and arrays. Is very useful to read a complete structure in only one line.

Is supported in node since ~v6.4.0, Chrome #49 and Firefox #41

Let's start!

### Array destructuring

In array destructuring each variable name on the array aps to the corresponding item at the same index on the destructured array.

This is our right side array.

```javascript
const bar = ["mate", "empanada", "milanesa", "asado"];
```

Back in time, we use a few lines to assign different items in an array
```javascript
const mate = bar[0];
const empanada = bar[1];
const milanesa = bar[2];
const asado = bar[3];
```

Now, we can do a single line assignment and destructuring
```javascript
const [mate, empanada, milanesa, asado] = bar;
```

Or simply skip values using comma (`,`) to mit and grab last value
```javascript
const [,,, asado] = bar;
```

or first and rest
```javascript
const [mate, ...rest] = bar;
```

we can use destructuring returning values
```javascript
const f = () => [1, 2];
const [uno, dos] = f();
```

or destrucutre in a for-of
```javascript
for (const [index, element] of bar.entries()) {
	// Process
}
```

Is important to remark here that we can use only rest operator as last parameter
```javascript
const [...rest, asado] = foo; // syntax error
```

Other use is to swap values in one line, without using an extra variable
```javascript
let playerOne = ‘Matt’
let playerTwo = ‘Alex’

[playerOne, playerTwo] = [playerTwo, playerOne]
```

### Object destructuring
For object destructuring we use an object in the left side of an assignment.
Our first example of object destructuring is to simplify module load. In this way, we only load the required modules
```javascript
const { empanada, asado } = require('@some/module');
```

We can use object destructuring to assign default values finstead if `undefined` for keys
```javascript
function f({message = '', number = 0}) {
	// message === ’’ --> true
	// number === 20 --> true
return result;
};

const parameter = {
	number: 20,
};

f(parameter);
```

For code simplicity, we only want one key with default value
```javascript
function f({message = 'default value'}) {
	// message === ‘mate con empanadas` --> true
	// number === undefined --> true
	return result;
};

const parameter = {
	number: 20,
	message: 'mate con empanadas'
};

f(parameter)
```

## Rename

Another use is to rename variables
```javascript
const { status: statusCode, statusText} = await request(URI, BODY, HEADERS);
```
