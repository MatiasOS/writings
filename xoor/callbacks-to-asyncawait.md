# From callbacks to async/await

I stared coding in Abap, with a very sequential and step by step reports where everything works as expected. A team of coworkers were working in an internal reporting app. As the workload diminished with Abap, I moved to the Javascript world, and that was my first `close encounter` with callback hell. 
I remember being lost among so many functions, doing console here, console there, why is this undefined?
That sounds familiar to you? Just keep reading!

## Callbacks
A callback is a function that is passed to a another function to be excecuted at some point. Ex:

```javascript
const callBackFunction = () => {
  console.log("I'm the callback!");
}

const functionWithCallback = (callback) => {
  console.log("I'm a function, I'll do some work, and then call the callback");
  // Do some stuff
  callback();
}

functionWithCallback(callBackFunction);
```

Usually, we use a library that uses callbacks
```javascript
import { functionWithCallback } from 'awesome-functions';

const callBackFunction = () => {
  console.log("I'm the callback!");
}

functionWithCallback(1, 2, callBackFunction);
```

We, the developers, are lazy. We love to use anonymous functions 
```javascript
import { functionWithCallback } from 'awesome-functions';

functionWithCallback(1, 2, () => {
  console.log("I'm the callback!"); // This first identation is where all starts
});
```

That harmless indentation is fine, but it gets worse and bigger when we need a sequence of asynchronous calls. Ex.
```javascript
/*
* We want to fetch users that know how to prepare a list of recipes`.
* For each recipe we want to call other API to get the ingredients. 
*/
fetch(USERS_API_URI, (err, user) => {
  if (error) throw error;
  
  user.recipes.forEach((recipe)=> {
    fetch(RECIPES_API_URI, (err, recipes) => {
      if (error) throw error; 
      // Do something with the list of ingredients here.
      // Add all the ingredients to the result
    });
  });
})
```

We can see problems here with readability, error handling and race conditions. 
We have `if (error) throw error;` inside each callback. This brakes [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
On the other hand , we don't really know wich recipe will come first because API calls depends on server loads and network traffic. 

We can use named functions for each callback to improve readability a bit
```javascript
const handleUserResponse = (err, user) => {
  if (err) throw err;
  const recipes = user.recipes; 
  recipes.forEach((recipe) => {
    fetch(RECIPES_API_URI,, handleRecipeResponse);
  })
}

handleRecipeResponse = (err, recipe) => {
  if (err) throw err;
  const ingredients = recipe.ingredients; 
  // Handle the repositories list here
  console.log(`The ingredients for ${recipe.name} are: ${ingredients}`);
}

fetch(USERS_API_URI, handleUserResponse);
```

We still have racing and error handling issues. This leads to maintenance and readability problems. 
That grows until code becomes unmanageable.

We reach a point where the code is mostly functions, callbacks, check if error exists. This is callback hell! also called pyramid of doom. I know! Sounds really tragic.

Callback Hell can be avoided with our next approach, Promises.

## Promises

With Promises, we can make our code more readable. Besides, it eases the understanding of the order of execution of our code.
A Promise is a proxy for a value, not necessarily known at the moment the promise is created. It allows you to associate handlers with an asynchronous action with an eventual success value or failure reason.

```javascript
const aPromise = new Promise((resolve, reject) => {
  const { response, error} = doSomething(); 
  if (error) {
    resolve('Promise resolved')
  } else {
    reject('Promise rejected')
  }
  
})

aPromise
  .then((response) => {
    console.log(response);
    return response;
  })
  .catch((err) => {
    console.error(err)
  });
```
Multiple callbacks may be added by calling `.then()` several times. Each callback is executed one after another, in the order in which they were inserted.

With a chain on `.then` if an error happens on the first one it will skip subsequent `.then` until it finds a `.catch`. 

Lets refactor our code using Promises
```javascript
fetch(USERS_API_URI)
  .then((user) => {
    Promise.all(user.recipes.map((recipe)=> {
      return fetch(RECIPES_API_URI);
    });
  }))
  .then((recipe) =>{
      console.log(`The ingredients for ${recipe.name} are: ${recipe.ingredients}`);
  })
  .catch((err) => {
     console.log('Something in the Promise chain had an error!', error)
  }) 
```

This is still a bit confusing still, lets use named functions instead of anonymous
```javascript
fetch(USERS_API_URI)
  .then(handleUsersApiResponse)
  .then(handleRecipesApiResponses)
  .catch(handleError) 

const handleUsersApiResponse = (user) => {
  Promise.all(user.recipes.map((recipe)=> {
    return fetch(RECIPES_API_URI);
  });
};

const handleRecipesApiResponses = (recipes) => {
  recipes.forEach(handleRecipe)    
}

const handleRecipe = (recipe) => {
  console.log(`The ingredients for ${recipe.name} are: ${recipe.ingredients}`);
}

const handleError = (err) => {
  console.log('Something in the Promise chain had an error!', err)
};
```
We have a better idea of this code block.

### Promises - Don't do

We need to be aware of the common mistakes to prevent the replication of bad practices 

#### Confusing code

We need to be sure to properly use Promises to prevent confusing code
```javaScript
doSomething().then(function(result) {
  doSomethingElse(result) // Forgot to return promise from inner chain + unnecessary nesting
  .then(newResult => doThirdThing(newResult));
}).then(() => doFourthThing());
// Forgot to terminate chain with a catch!
```

#### Unnecessary nesting

If we don't chain things together properly, we need nesting to hanlde code dependecies

```javascript
db.findAll().then((result) => {
  const docs = result.rows;
  docs.forEach((element) => {
    otherDb.put(element.doc).then((response) => {
      logger("Pulled doc with id " + element.doc._id + " and added to local db.");
    }).catch((err) =>{
      if (err.name === 'conflict') {
        otherDb.get(element.doc._id).then((resp) => {
          otherDb.remove(resp._id, resp._rev).then((resp) => {
            // It start to looks like a Promise Hell  
```

A good rule-of-thumb is to always either return or terminate promise chains, and as soon as you get a new promise, return it immediately
```javascript
db.allDocs()
.then((resultOfAllDocs) => {
  return otherDb.put(...);
})
.then((resultOfPut) => {
  return otherDb.get(...);
})
.then((resultOfGet) => {
  return otherDb.put(...);
})
.catch(function (err) => {
  logger(err);
});
```

#### Is everything resolved? 

Other common error is believe that all Promises have been resolved!
```javascript
// I want to remove() all docs
db.allDocs({include_docs: true}).then(function (result) {
  result.rows.forEach(function (row) {
    db.remove(row.doc);  
  });
}).then(function () {
  // We naively believe all docs have been removed() now!
});
```

We should use `Promise.allSettled` or `Promise.all`. Ex.
```javascript
db.allDocs({include_docs: true}).then(function (result) {
  return Promise.all(result.rows.map(function (row) {
    return db.remove(row.doc);
  }));
}).then(function (arrayOfResults) {
  // All docs have really been removed() now!
});
```

## Async/await

Promises paved the way to one of the coolest improvements in JavaScript. ECMAScript 2017 brought in syntactic sugar on top of Promises in the form of `async` and `await`.
They allow us to write Promise-based code as if it were synchronous.

Declaring a function as async will ensure that it always returns a Promise so we don’t have to worry about that anymore.

We define our functions as `async` and what part of the code will have to `await` for that `promise` to finish.

An async function can contain an `await` expression that pauses the execution of the async function to wait for, then resumes the async function's execution and evaluates as the resolved value.

Having the code well structured with Promise, is really easy and fast to move to `async/await`. 

We could use the synchronous 
```javascipt 
  try {
    ...
    } catch(e) {
      ...
    }
``` 
structure with `async/await` to handle errors. Let's do it!
```javascript
try {
  const user = fetch(USERS_API_URI);
  users.recipes.forEach(async (recipe) => {
    const fetchedRecipe = await fetch(RECIPES_API_URI, recipe);
    handleRecipe(fetchedRecipe);
  });
 } catch (err) {
    console.log('Something went wrong!', err);
 }

const handleRecipe =  (recipe) => {
  console.log(`The ingredients for ${recipe.name} are: ${recipe.ingredients}`);
}
```

But maybe we don't want to wait for this fetch 
```javascript
const recipes = await fetch(RECIPES_API_URI);
```

We still could use `Promiese.all` to wait in parallel. 
```javascript
  const recipePromises = user.recipes.map((user)=> fetch());
  const recipes = Promiese.all(recipePromises);
```

### Async/await - Don't do

#### Blocked execution

Our code could be slowed down by awaited promises. Each await will wait for the previous one to finish, whereas actually what we want is for the promises to begin processing simultaneously.
```javascript
async function (param) {
  const r_1 = await f1(param);
  const r_2 = await f2(param);
  const r_3 = await f3(param);
  const r_4 = await f2(param + 1);
  const r_5 = await f3(param + 1);

  // Do something with all the results
}
```

All these statements execute one by one. There is no concurrency here. A better approach is to don't `await` for 
```javascript
async function (param) {
  const r = Promise.all([
    f1(param),
    f2(param),
    f3(param),
    f2(param + 1),
    f3(param + 1),
    ]);
  // Do something with results
}
```

## Final words

* Should you start using the JavaScript async function today? Yes, we should! Why?
  - The resulting code is much cleaner.
  - Error handling is much simpler and it relies on try/catch just like in any other synchronous code.
  - Debugging is much simpler. Setting a break-point inside a `.then` block will not move to the next `.then` because it only steps through synchronous code. But, you can step through await calls as if they were synchronous calls.

* Wrap parallel code in `Promise.all` or `Promise.allSettled` 
  * If some code is meant to be run in parallel, we should not make it sequential. 
  * We can now use array manipulation functions like `map`, `reduce` and `forEach`.
  * There is also `Promise.race` for cases where we need to wait for any one of the methods to respond and do not need to wait for all of them to continue execution.

* Leave the side effects without `await`
  * Sometimes we do have side effects like leaving a call to analytics that we do not wish to wait for or logging that can happen after the response is sent from the server. For these, we can ignore the await call and it works just like before.

* Some code can still have callbacks. 
  * Not all callbacks need to be converted. Methods like `addEventListener` are still callback-based and are natural to remain that way. We don't have sequential flow of control that the callback-based code was making difficult to understand, there is no point in removing callbacks.


## References
* [MDN Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [MDN Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
* [MDN Async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* [MDN await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
* [Hello JS](https://blog.hellojs.org/asynchronous-javascript-from-callback-hell-to-async-and-await-9b9ceb63c8e8)
* [DZone](https://dzone.com/articles/from-callbacks-to-async-await-a-migration-guide)
* [FreeCodeCamp article](https://www.freecodecamp.org/news/javascript-from-callbacks-to-async-await-1cc090ddad99/)









