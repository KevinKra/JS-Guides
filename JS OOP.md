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
 - `retroactive inheritance`
 - `syntactic sugar`
 - `prototype nature`
 - `prototype chain`
 - `implicit variable`
 - `subclassing`
 - `Factory Function`
 - `this` and `this`
 - `new`
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

```
var a = 5;
var b = a;
a = 10;
var a;
var b;
b += 15;
```
```
var apple = [1,2,3]
var toast = apple;
apple.push(3);
apple;
toast;
```
The first example is passing by value, we are literally copying the data and assigning it to a new place in memory. Another way to think about it is: `var b` equals the value stored in `var a`, once that initial assignment is completed, the bond is broken. This method of passing values is limited to `primitives`.

The second example pertains to complex data structures. Objects, and their respective subtypes of arrays and functions, all reference a specific place in memory. They essentially all point to the same location. When you declare a variable and assign it to an array (like above), the variable never truly *holds*  the value, but rather references it. When we assigned the value of `apple` to `toast` we were more accurately giving toast the same reference to the array in memory.

To simplify the concept, image copying to be akin to `var a` being a person holding a pineapple, `var b` says I want to be exactly like `var a`, so I too am going to hold a pineapple. Link severed. `var a` now decides to hold a tomato. `var b` no longer cares about what `var a` is up to, so he simply keeps holding onto his pineapple. Passing value by reference is even more straightforward, everyone points to the basket of fruit, if it's changed everyone recognizes the modification.

*I used these two examples to illustrate different ways to think about how data gets passed around, we will see that the linkages made between objects are neither passing by value or reference, but conceptually it mirrors the design of passing by reference in the sense of n objects all pointing to the same place.*


> these concepts are very important in regards to functional programming and data mutation. A topic for a future guide. Just remember, in the event you need to mutate a data structure copy it and transform the copy, not the original that everyone is referencing.

**Ok, back to Objects**
Our above example, while it did expedite the object creation process, it had the fundamental issue of creating new objects with unique data (good) that also included needless copies of methods (naughty). Wouldn't it be nice if there was one place in memory where all our objects could point to and simply store the shared functionality there?

we can do this manually with `Object.create(null)`

>Lets discuss this native Object method. 

`let a = Object.create(null);`
In the above example, we are using `Object.create()` to create an empty object, to visualize: `a = {}`. 
>So uh, cool but what's the point of this? 

Great question, try assigning a variable to `Object.create()`, you will receive a TypeError. This method comes with a very important feature that is determined by what we pass in as an argument. Let's look at an example that illustrates this feature.

```
const userFunctions = {
    passTime : function() {
        this.age++;
    },
    logIn : function() {
        console.log('Beep boop logged in.');
    }
};

function UserCreator(name, age) {
    const newUser = Object.create(userFunctions);
    newUser.name = name;
    newUser.age = age;
    return newUser;
};
```
>So what's going on here?

We created an **object** with the identifier `userFunctions`, this **object** contains several methods inside of it. When we declare out `UserCreator` function, inside its function declaration we assign the value of `Object.create(userFunctions)` to the constant newUser. 
> So how does it actually link to the object we provided as an argument??

`__proto__`, aka "dunder proto" (double underscore proto). 
>  alternate: `object.getPrototypeOf()` since `__proto__` is a private variable and becoming phased out.

`__proto__` is the link that connects our instantiated objects to a shared reference point in memory. In this case we used the Object method `.create()` with an argument object type. This object serves as the (in this case explicitly) as the storage location for our shared functions.
>This is great. We have a way to rapidly create objects and dynamically assign their key (property) value pairs. Additionally, we also have a solution for our performance issue: we have a nice tidy object that can house all of our functionality and every instanced object can reference the same object. 
>One issue, I don't like typing all that out... isn't there some way we can automate it? 

**Introducing new, the other *this*, and prototypes!**

Let's address the *new* elephant in the room. The **new** keyword. One may be lulled into thinking this is just a nice fancy word that just helps us in some minor way while we go about instantiating our objects. One would be wrong. The `new` keyword, or as Getify calls it the "new constructor call", is a very powerful keyword that does a lot of stuff under the hood for us.

Let's look at this example
```
function Hobbit(name, age, hungry) {
    this.name = name;
    this.age = age;
    this.hours = 0;
    this.hungry = hungry;
} 

let hobbit1 = new Hobbit('Frodo', 94, false);
```
>starting to look familiar?

 1. the `new` keyword creates an empty object named **this** out of thin air.
 2. the `new` keyword creates a `__proto__` link to the `Hobbit`'s prototype.
 3. the `new` keyword assigns the this's from the *this-aware* function `Hobbit`
 4. the `new` keyword automatically returns the function.

**Wait, so is this `this` like the other `this`?**
Don't worry, this isn't confusing at all. Okay, maybe a little ... at first. There are two `this`'s in Javascript. There is a `this` object that is automatically created by the `new` keyword, assigned accordingly, and then garbage collected when the function completes its course. The second and more prevalent `this` is oriented around object binding, or context objects, as you'll see in a second.

`this` as a context object using explicit binding rules.
```
let obj1 = {
    score : 0,
    increment : function() {
        this.score++;
    }
}

let obj2 = {
    score: 15
}

obj1.increment.call(obj2);
```


> Also, what about that prototype thing you just mentioned?

Fear not, here it is.

```
Hobbit.prototype.isHungry = function() {
    this.hours++;
    this.hungry = ( this.hours > 2 );
}
```
Now I want to talk about something very interesting about JS, something that has been bouncing in my mind for a while regarding the language. Getify coined (I assume) the term `OLOO` *Objects Linked to Other Objects*, he said it is a more accurate representation of what is actually happening in JavaScript. In traditional Object Oriented languages like Java and C, a blueprint or genetics metaphor is used to explain the process of object instantiation. In short, after an object is instantiated from the constructor, their link is effectively severed. 

>we just did something very different and unusual.

In traditional `OO` languages after we declared `hobbit1` and used the `new` constructor call we would have essentially severed the link to the constructor function. However, we were still able to add the method `isHungry` to our constructor function's prototype and use it in our previously created function. This is called `retroactive inheritance`. Hopefully this drives the point home that JavaScript is not like traditional `OO` languages, it is instead linked together behind the scenes. 

>Don't be fooled by the `class` implementation that ES6 introduced, while it's simpler and easier to reason about it is called `syntactic sugar` for a reason.

**the Class construct**

```
class Hobbit {
    constructor(name, age) {
        this.name = name;
        this.age = age;
        this.hours = 0;
        this.hungry = false;
    }
    isHungry() {
        this.hours++;
        this.hungry = (this.hours > 2);
    }
}

let hobbit1 = new Hobbit('Greg', 99);
```
If we peel back the layers of this Class onion, we would see that it actually is built exactly the same as the above prototype approach seen earlier. For instance, if we `console.log(Hobbit.prototype)` we would find `isHungry` method actually living on that property object. Neat.

> So why introduce classes at all? The previous prototype approach was more explicit, this class system is deceptive and abstracts away a lot of what made the prototype approach helpful. The answer is that it copies the structure that is prevalent in a lot of other languages, many people learning Javascript after learning Java were frustrated with the JS implementation to object instantiation. So, they introduced something that *looks* like the traditional `OO` class construct, but we now know (hopefully) how it is fundamentally different.

**JS's `prototype nature` and `prototype chain`**
```
const a = {
    name : 'steven'
}

a.name //steven
a.hasOwnProperty('name') 
```
Javascript uses its prototype nature to look up the prototype chain and finds the `hasOwnProperty` method located on the built-in (or native) `Object` "function-object combo". *Capital O* `Object`, along with several other capital lettered function-objects, come with the language. When we are unable to find the relevant methods on our immediate object, Javascript follows the `__proto__` linkage until it eventually finds the relevant called method or property.   

In the above example, `hasOwnProperty` method was found on the `Object`'s `prototype` object property.


**Subclassing**
>Ordering the distribution of our functions among our linked objects in a structured way 

By using subclassing we are able to create more specific classes that can utilize the functionality and data of previous classes while preventing the previous classes from accessing our more privileged functionality and data.

lets look at the explicit subclassing example below.
```
//we create a normalUser factory function and __proto__ link it to the userFunctions Object.
function normalUser(name, score) {
    const newUser = Object.create(userFunctions);
    newUser.name = name;
    newUser.score = score;
    return newUser;
}

const userFunctions = {
    logOut : function() {console.log("Beep boop logged out")}
}

//we then create a new factory function that creates more privileged objects than before.
function paidUser(paidName, paidScore, accountBalance) {
    const newUser = normalUser(paidName, paidScore);
    Object.setPrototypeOf(newUser, paidUserFunctions);
    newUser.accountBalance = accountBalance;
    return newUser;
} 

const paidUserFunctions = {
    increaseBalance : function() {
        this.accountBalance += 500;
    }
}

Object.setPrototypeOf(paidUserFunctions, userFunctions);

let a = normalUser('Kevin', 250);
let b = paidUser('Toby', 550, 2500);
```

Now let's repeat this process with the `new` keyword, it will require us to use binding rules to establish the original function object as the context object for our subclassing function.

> the *this* in the `normalFridge.call(this ...)` will be the auto-created `this` object created by the `new` keyword in `myFancyFridge = new fancyFridge(...)`. We are using that `this` and passing in our fancyFridge's fancyName and fancyCost arguments to utilize the normalFridge's `this` assignments, but we are returning out the new fancyFridge `this` (automatically) when we instantiate it with `let myFancyFridge = new fancyFridge('iFridge', 5000);`

```
function normalFridge(brand, cost) {
    this.brand = brand;
    this.cost = cost;
    this.on = false;
}

normalFridge.prototype.turnOn = function() {
    return this.on? console.log(`Your ${this.brand} is already activated. ☜(꒡⌓꒡)`) : this.on = true;
}


function fancyFridge(fancyName, fancyCost, fancyMagnet) {
    normalFridge.call(this, fancyName, fancyCost);
    this.fancyMagnet = fancyMagnet;
    this.totalMagnets = 1;
}

fancyFridge.prototype = Object.create(normalFridge.prototype);

fancyFridge.prototype.brag = function() {
    this.totalMagnets++;
    console.log(`Yeah I have a fancy ${this.brand} and have um, ${this.totalMagnets} limited edition magnets.`);
}

let myFridge = new normalFridge('Lenovo', 500);
let myFancyFridge = new fancyFridge('iFridge', 5000);
```

**ES6 (sugary) Class approach**
> our base class

```
class Vampire {
    constructor(name, age, thirsty = false) {
        this.name = name;
        this.age = age;
        this.thirsty = thirsty;
        this.human = true;
        this.bat = false;
    }
    xform() {
            this.human = !this.human;
            this.bat = !this.bat;
    }
    toBat() {
        return this.human ? this.xform() : `I'm already a bat!`;
    }
    toHuman() {
        return !this.human ? this.xform() : `I'm already a human!`;
    }
}


var person1 = new Vampire('Andre', 27);

```

>let's extend (our subclass)

```
class SpecialVampire extends Vampire {
    constructor(specialName, specialAge, glitter = false)   {
        super(specialName, specialAge);
        this.glitter = glitter;
        this.alive = true;
    }
    directSunlight() {
        if (this.alive) {
            this.glitter = true;
            this.alive = false;
            console.log(`ah damn the sun ouch`);
        } else {
            console.log(`I'm not in direct sunlight`);
        }
    }
}

var person2 = new SpecialVampire('Edward', 187);
```


//Still under construction
Things that **extend** **super** do:

1. `extend` sets the `__proto__` link of SpecialVampire's `prototype` to the `prototype` of the Vampire function/object.
2. `super` uses constructor function in Vampire, creates the object there, and then sets the `__proto__` 
3. `extend` sets the `__proto__` linkage of SpecialVampire (not it's `prototype`) to Vampire allowing `super` to create an object inside of Vampire and carry it's own specialName and specialAge arguments. 
4. `this` created by the `new` keyword in `var person2 = new SpecialVampire('Edward', 187);` is initialized, but assigned in Vampire and returned out. 




