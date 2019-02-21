# ES6

## Let, Const, Var
ES6 introduced a new form of scope known as **block-level scope** or *block scope*. Until the addition of block scope, the only way to isolate and contain variables in their own respective variable environments one had to create functions and wrap all the relevant data inside the function's body. However, there were still lingering issues that needed to be addressed.

**Block Scope**

*ES5* 
```
function foo() {
    var i = 5;
    for(var i = 0; i < 3; i++) {
        console.log('something');
    }
    console.log(i) //?
}
```

*ES6 block scope*
```
function foo() {
    var i = 5;
    for(let i = 0; i < 3; i++) {
        console.log('something');
    }
    console.log(i); //?
}
```

**Differences between let, const, and var**

- **var** can "hoist", or more accurately, it is assigned to its respective scope during the compilation phase and then initialized during run time when its lexical scope container is entered. As far as the interpretor is concerned, var *does* exist, but until the line is reached were the variable is assigned a value it will remain undefined.
- **let** and **const** do not hoist. They are assigned scope, but not initialized.
- **let** can be re-assigned, but *not* redeclared.
- **const** cannot be re-assigned *or* redeclared.
- **!important** while **const** cannot be re-assigned, if its value is a non-primitive type it is **mutateable!**

> let's look at some examples

*var*
```
a //undefined
var a = 5;
var a = 10;
a = 15;

a //?
```

*let*
```
a             //ReferenceError 'a' is not defined
let a = 5;
let a = 10;   //SyntaxError 'a' has already been declared
a = 15;

a //?
```

*const*
```
a             //ReferenceError 'a' is not defined (should say declared)
const a = 5;
const a = 10; //SyntaxError 'a' has already been declared
a = 15        //TypeError assignment to constant variable

a //?
```

*const pt.2*
```
const a = [1,2,3]

const a = [5,6,7] //error

... but 

a.push(5)         // a = [1,2,3,5]
```

>>Again, while you cannot re-assign a constant with an entirely new value, you can mutate complex data structures even if they are assigned to a constant declaration. Explore deepfreeze to prohibit that feature.

## Arrow => Functions

Arrow functions allow developers to significantly decrease the amount of typing required write a function. Additionally, they also help resolve certain binding issues where the *this* context object would otherwise be lost.

*ES5*
```
var addBy5 = function addBy5(number) {
    return number + 5;
}
```

*ES6*
```
const addBy5 = number => number + 5;
```

Few things to note:
1 - arrow functions are *function expressions*.
2 - if there is a single parameter, parenthesis not required.
3 - No brackets no return statement needed (it implicitly returns)

>let's run through a few more interesting examples 

**Two parameters**
*ES5*
```
var multiplyTwoNumbers = function(num1, num2) {
    return num1 * num2;
}
```

*ES6*
```
const multiplyTwoNumbers = (num1, num2) => { return num1 * num2 }
```
>brackets and return statement are optional here, but in a situation where you need multiple lines for your statements, brackets are required.

**ES6 shorthand (no parameters)**
```
const sayHello = () => console.log("hello");

// or even

const sayHello = _ => console.log("hello");
```

**ES6 returning an Object literal**
```
const shareMyObject = () => ({apples : 'yum'});
```
>Objects being returned require parentheses

## Destructuring

