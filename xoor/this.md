# This

First of all, watch this [video](https://youtu.be/_NNYI8VbFyY), and then you can continue reading.\
I remember a long time ago struggling with `this`, `bind`, `self = this` and several related problems. But in some ponit, I found a post with that [video](https://youtu.be/_NNYI8VbFyY) that help me a lot to undestand this problem. If you know the post, please leave the link in comments!

To understand this problem, we must know that
```
The this keyword does not refer to the function in which it is used or it’s scope. It refers to the object on which a function is being executed and depends entirely on the call-site of the function.
```
\
Javascript has different ways to bind `this`. We'll analize the differences in the next sections.

## Implicit binding

```javascript
const food = {
  kind: 'schnitzel',
  WhatIAm() {
    console.log(`Hi! I'm a ${this.kind}`);
  }
}
food.WhatIAm() // Prints -> Hi! I'm a schnitzel
```

But, if we create a new reference to this function

```javascript
const WhatIAmHere = food.WhatIAm
WhatIAmHere(); // Prints -> Hi! I'm a undefined
```
`this.kind` is undefined! It's beacuse, `this` points to a global object (window in the browser, and process in node).

Here, is defined on the `food` object. But we need to look where it is called.  A trick is to look at the left side of the function call. The object that is standing before the dot is what the `this` keyword will be implicitly bounded to.

## Explicit binding

We need some way for explicitly bindining. Those ways are `call`, `apply`,  `bind` and `new`.

### Call

The `.call` function allows you to pass in the object to which the `this` keyword should be bound to. It also allows you to pass the arguments for the function.

```javascript
function WhatIAm(arg1, arg2, arg3, ...) {
  console.log(`Hi! I'm a ${this.kind}`);
}

const food = {
  kind: 'milanesa'
};

WhatIAm.call(food, arg1, arg2, arg3, ...); // Prints -> Hi! I'm a milanesa
```
`this` is poiting to the `food` object because is passed as first parameter..

### Apply

The `.apply` function is similar to `.call` with the difference that the function arguments are passed as an array (or an array-like object).

```javascript
function WhatIAm(args) {
  console.log(`Hi! I'm a ${this.kind}`);
}

const food = {
  kind: 'milanesa'
};

WhatIAm.call( food, [arg1, arg2, arg3] ); // Prints -> Hi! I'm a milanesa
```
`this` points to the `food` object.

### Bind 

The `.bind` function is a little bit different than the first two. It creates a new function that will call the original one with `this` bound to whatever was passed as parameter.

```javascript
function WhatIAm() {
  console.log(`Hi! I'm a ${this.kind}`);
}

const food = {
  kind: 'milanesa',
};

const whatKindOfFood = WhatIAm.bind(food);
whatKindOfFood(); // Prints -> Hi! I'm a milanesa
```

### New Keyword

Whenever you invoke a function with the `new` keyword, under the hood, the JavaScript interpreter will create a brand new object for us and call it `this`. If a function was called with `new`, `this` keyword is referencing that new object.

```javascript
function food (kind) {
  this.kind = kind;
}

const myFood = new food('empanada');
console.log(myFood.kind); // Prints -> empanada
```

### Arrow functions and lexical scope

Arrow functions(`() => { ... }`) use lexical scoping, it means that it uses `this` from the site that contains the Arrow Function. In other words, the scope is determined when the code is compiled.

```javascript
const food = {
  kind: 'empanada',
  whatIAm: whatIAm () => {
    console.log(`Hi! I'm a ${this.kind}`);
  },
}

this.kind = 'milanesa';
food.whatIAm(); // Prints -> Hi! I'm a milanesa
```
This happens because `this` isn't referencing `food` object. 

## Conclusion

As final words, I want to show a few scenarios where is commonly used:
In React, sometimes we need to access `this.state`, and it can be tricky

```javascript
class ui extends Component {
  render() {
    const self = this;
    someArray.map(function(e) => {
      return (<SomeComonent 
        value={self.factor * e.value}
      />
      );
    });
  }
}
```
A good way to prevent this bad practice, is to use an arrow function
```javascript
class ui extends Component {
  render() {
    someArray.map(({id, value}) => (
      return (<SomeComonent key={id} value={this.factor * value} />);
    ));
  }
}
```
Another common place for this kind of problems is handlers methods

```javascript
class ui extends Component {
  handleClick = (event) => {
    this.setState({});
  }

  render() {
    return (<SomeComonent onClick={this.handleClick} />);
  }
}
```
the function `this.handleClick` becomes the handler. Under the hood, roughly the following code happens

```javascript
const handler = this.handleClick;
handler(); // or handler.call(undefined);
```
And because of this, we ussually bind functions

```javascript
class ui extends Component {

  constructor() {
    handleClick = this.handleClick.bind(this);
  }
  handleClick = (event) => {
    this.setState({
      // ...
    });
  }

  render() {
    return (<SomeComonent onClick={this.handleClick} />);
  }
}
```

To finish, I found this rules to understand better  what `this` is referencing: 

1. Look to where the function was invoked.
2. Is there an object to the left of the dot? If so, that’s what the “this” keyword is referencing to. If not, continue to #3.
3. Was the function invoked with “call”, “apply”, or “bind”? If so, it’ll explicitly state what the “this” keyword is referencing to. If not, continue to #4.
4. Was the function invoked using the “new” keyword? If so, the “this” keyword is referencing the newly created object that was made by the JavaScript interpreter. If not, continue to #5.
5. Is “this” inside of an arrow function? If so, its reference may be found lexically in the enclosing (parent) scope. If not, continue to #6.
6. Are you in “strict mode”? If yes, the “this” keyword is undefined. If not, continue to #7.
7. JavaScript is weird. “this” is referencing the “window” object.

## References
* [2ality: This](http://2ality.com/2014/05/this.html)
* [2ality: Alternate this](http://2ality.com/2017/12/alternate-this.html)
* [exploringjs](http://exploringjs.com/es6/ch_callables.html)
* [Understanding the "this" keyword, call, apply, and bind in JavaScript](https://tylermcginnis.com/this-keyword-call-apply-bind-javascript/)
* [Alexander Kondov post](https://hackernoon.com/understanding-javascript-the-this-keyword-4de325d77f68)
* [Tyler Mc Ginnis post](https://tylermcginnis.com/this-keyword-call-apply-bind-javascript/)