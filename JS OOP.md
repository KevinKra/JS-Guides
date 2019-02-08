# JavaScript OOP


> *what is really happening under the covers*

**Vocabulary**

 - `encapsulation`
 - `instantiation`
 -  `method`
 - `property`
 - `copy` vs `reference`
 - `__proto__`
 - `prototype`
 - `constructor`
 - `implicit variable`
 - `this` and `this`
 -  `new`
 - `OOLO` vs `OO`

 



**Objects**
let's start at the top, what is an object and why use one.

Create an object literal:
```
const user1 = {
    name : "phil",
    age : 22,
    logIn : function() {
        console.log('beep boop, logged in.')
    }
}
```

Issue with object literals:

 - It's a lot of work typing all this out. It's not really an issue if we have one object, but what if we want to create 100+ objects from this template? Hmm.


> There must be a better way... perhaps something fundamentally designed to save us from repeating ourselves...

**Let's use a function!**
```
function UserCreator(name, age) {
    let newUser = {};
    newUser.name = name;
    newUser.age = age;
    newUser.passTime = function(){
        newUser.age++;
    }
    return newUser;
}
```
In the above example we used a function to solve the problem of having to write out  our object every time we want to create a new instance of it. 

 1. We declare a function `UserCreator`  with two parameters. Make sure the first letter is capitalized to signify it's a constructor function, more on this later.
 2. Inside the body of the function we create an empty object literal and assign it to `newUser`.
 3. We create properties on this empty object and assign them the values (arguments) passed into our parameters.
 4. We assign a function value to the key `passtime`, effectively making it a method on the `newUser` object.
 5. We return the value held in `newUser`, which happens to be an object. The variable `newUser` gets garbage collected as we exit execution context of the `UserCreator` function.

>Issue with this format?

While we are able to (more rapidly) create copies of our object, there is still a very troubling performance issue. Imagine a scenario where we use the above solution to instantiate our object, if we had 5 methods on the object and we have 5,000 users, now we have repeated (copied) the methods 25,000 times.

>There must be a better way… perhaps something fundamentally designed to save us from repeating our methods…

**What if we there was a place we could hold the shared functionality...**

Before we delve into this solution, let's discuss the two ways of passing values around in JavaScript. Passing by `reference` and passing by `value`.
...