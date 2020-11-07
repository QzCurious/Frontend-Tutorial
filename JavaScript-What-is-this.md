# JavaScript What is `this`

## What do we do with `this`

_In OOP, `this` is a reference to current object._

There is a few ways to create objects in JavaScript:

```javascript
// class
class C_Person {
    constructor(name) {
        this.name = name;
    }
    my_name_is() {
        return this.name;
    }
}
var c_person = new C_Person("c_person");
c_person.my_name_is(); // 'c_person'

// Constructor function, should only be call with `new`
function F_Person(name) {
    // this = {};  (implicitly)
    this.name = name;
    this.my_name_is = function () {
        return this.name;
    };
    // return this;  (implicitly)
}
var f_person = new F_Person("f_person");
f_person.my_name_is(); // 'f_person'

// object literal
var person = {
    name: "person",
    my_name_is: function () {
        return this.name;
    },
};
person.my_name_is(); // 'person'
```

Usually, we'll use class or constructor function to make objects.

## Scope

You(your code) are at somewhere and **what you can see**. You can only look(access) around or outward.

### local scope vs global scope

variables in a function has a local scope to the function. everything else are global scoped.

```javascript
var outside = "outside";
var overlap = "overlap outside";

function f() {
    var inside = "inside";
    var overlap = "overlap is redifined inside local scope";

    console.log(overlap); // 'overlap is redifined inside local scope'
    console.log(inside); // same level
    console.log(outside); // outter level
}

if (true) {
    var still_under_global_scope = "still_under_global_scope";
}

f();
console.log(overlap); // 'overlap outside'
console.log(inside); // Uncaught ReferenceError: inside is not defined
console.log(outside); // same level
console.log(still_under_global_scope);
```

### Lexical Scope

Inner function can access outer variables.

```javascript
function outer() {
    var where = "outside";
    console.log(`outer: ${where}`);
    function inner() {
        console.log(`inner: ${where}`);
    }
    inner();
}
outer();
```

Create a static-variable like variable. A cool example from [JAVASCRIPT.INTO](https://javascript.info/closure#nested-functions)

```javascript
function makeCounter() {
    let count = 0; // it's just like static variable cool!

    return function () {
        return count++;
    };
}

let counter = makeCounter();

console.log(counter()); // 0
console.log(counter()); // 1
console.log(counter()); // 2
```

## Block Scope

_Another store for `let` vs `var`_

## Context

`this` is the keyword refer to the context

### Global object

```javascript
this === window; // true
```

Global variables and functions lives under global object, `window`, in browser

```javascript
var foo = "foobar";
foo === window.foo; // Returns: true
```

### `this` is not always this

`this` is the master, is who owns you. So by default, `this` is `window` (global object).

Function can be pass around, and can be a **member** of an object. Then, `this` changes.

> Analogy:
>
> Function is like ability. Everyone is able to tell who he/she is, but with his/her own name. (Example below)

_`this` is not binded to a function._

```javascript
this === window; // true

function f(is_this) {
    console.log(this === is_this);
}
f(window); // true

var master = { f: f };
master.f(window); // false
master.f(master); // true
```

### We lose `this`?!

A callback function is a function that passed as an argument of a function call.

A **method** is a function that belongs to an object, and because of it, a method has a master.

What if a method is treated as a callback function?

```javascript
// sidenote: window.name is special and could be vulnerable (XSS)
var name = "Global";
function who_you_are(name) {
    if (typeof name === "string") console.log(name);
    else if (typeof name === "function") console.log(name());
}

f_person.my_name_is(); // 'f_person'
who_you_are(f_person.my_name_is); // 'Global'
```

### Fix this to `this`

[`bind()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) can be used to bind context, that is, fix `this`. `bind()` return a function with fixed context.

```javascript
var binded_my_name_is = f_person.my_name_is.bind(f_person);
who_you_are(binded_my_name_is); // 'f_person'
```

#### Arrow function does not have its own bindings to `this`

```javascript
var name = "Global";

var arrow_function = () => {
    console.log(this.name);
};
arrow_function(); // 'Global'
arrow_function.bind(f_person)(); // 'Global'

function normal_function() {
    console.log(this.name);
}
normal_function(); // 'Global'
normal_function.bind(f_person)(); // 'person'
```

BUT! In the mean while, arrow function is convenient. Because it dose not have `this` (has no context), accessing `this` in arrow function falls back to the outer scope.

> MDN:
>
> The arrow function does not have its own this. The this value of the enclosing lexical scope is used; arrow functions follow the normal variable lookup rules. So while searching for this which is not present in the current scope, an arrow function ends up finding the this from its enclosing scope.

Let us rewrite [example](#What-do-we-do-with-this) with arrow function:

```javascript
// Constructor function, should only be call with `new`
function F_Person(name) {
    // this = {};  (implicitly)
    this.name = name;
    this.my_name_is = () => {
        return this.name;
    };
    // return this;  (implicitly)
}
var f_person = new F_Person("f_person");

f_person.my_name_is(); // 'f_person'
who_you_are(f_person.my_name_is); // 'f_person'
```

**Pitfall**: A literal object does not create local scope. In this case, `this` is fell back to the `this` under global scope

```javascript
// object literal
var name = "Global";
var person = {
    name: "person",
    my_name_is: function () {
        return this.name;
    },
};
who_you_are(person.my_name_is); // 'Global'
```

## Button line

For JavaScript:

-   > JavaScript **does not have dynamic scope.** It **has lexical scope**.

-   `this` is determined at runtime. So it's dynamic.
-   Scope is determined at [author-time](https://stackoverflow.com/questions/35686903/author-time-vs-run-time-in-javascript/38675011). So it's static.

## References

-   [Scopes in Javascript](https://towardsdatascience.com/still-confused-in-js-scopes-f7dae62c16ee)
-   [Global object](https://developer.mozilla.org/en-US/docs/Glossary/Global_object)
-   [Constructor, operator "new"](https://javascript.info/constructor-new)

-   [Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

-   [Javascript — Lexical and Dynamic Scoping?](https://medium.com/@osmanakar_65575/javascript-lexical-and-dynamic-scoping-72c17e4476dd)
-   [Variable scope, closure](https://javascript.info/closure)

## To do

This raise an error instead of fall back `this` to global object

```javascript
who_you_are(c_person.my_name_is);
```
