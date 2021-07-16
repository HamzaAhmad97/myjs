## Primitive and reference types:

Generic object:

```
let object1 = new Object();
```

other built in objects:

```
let colors = new Array("red", "blue", "green")
let now = new Date();
let error = new Error("Something bad happened.");
let reflect = new Function("value", "return value;");
let object = new Object();
let re = new RegExp("\\d+");
```

also using the literal method:

the better way to check object types since except for functions, using typeof will return object for all other object types, so it is better to use instanceof like this:

```
items instanceof Array // true
```

we use the instructor after the operator
The instanceof operator can identify inherited types.

```
items instanceof Object // true
```

when you pass an array from one frame to another, instanceof doesn’t work because the array is actually an instance of Array from a different frame/context.
the best way to identify arrays is:

```
Array.isArray(arr)
```

this is what happens behind the scenes when dealing with primitive types and using thier methods, primitive wrapper types get created and dereferenced / destroyed when done (the code actually creates a new property on a temporary object that is then destroyed):

```
let name = "Nicholas";
let firstChar = name.charAt(0);
console.log(firstChar);
```

is actually:

```
let name = "Nicholas";
let temp = new String(name);
let firstChar = temp.charAt(0);
temp = null;
console.log(firstChar);
```

also

```
console.log(name instanceof String); // false
```

and

```
console.log(typeof name); // object
```

so even this will work since an object is a truthy value:

```
let found = new Boolean(false);
if (found) {
console.log("Found");
}
```

---

## Functions:

the defining characteristic of a function—what distinguishes it from any other object—is the presence of an internal property named **[[Call]]** .
Internal properties are not accessible via code but rather define the behavior of code as it executes.These internal properties are indicated by double-square-bracket notation.

In function expressions functions are considered anonymous because the
function object itself has no name.

**arguments** is an array-like object (but not an instance of Array), but it behaves just like any other array, and it is always available inside all functions.

Named parameters are not ignored.The _number_ of arguments a function expects is stored on the function’s **length** property (since a function is just an object, this is one of its properties). The length property indicates the function’s arity, or the number of parameters it expects.

With named parameters:

```
function reflect(value) {
return value;
}

console.log(reflect.length); //1
```

Using arguments:

```
reflect = function() {
return arguments[0];
};
```

Since functions in js do not have any issues with accepting as many parameters as you like, we lose the ability to _overload_, and what happens behind the scenes is that when you define multiple functions with the same name, the one that appears last in your code wins, even if they differ in the number and types of parameters. The earlier function declarations are completely removed, and the last is the one that is used. In other words and using the constructor notation, we are basically changing the object reference for the same variable holding the function name. Still, we can mimic overloading by making use of the arguments and length properties.

In practice, checking the named parameter against undefined is more common than relying on arguments.length .

### **this**?

Every scope in JavaScript has a this object that represents the calling object for the function. In the global scope, this represents the global object ( window in web browsers). When a function is called while attached to an object, the value of this is equal to that object by default.

```
function sayNameForAll() {
console.log(this.name);
}

let person1 = {
name: "Nicholas",
sayName: sayNameForAll
};

let name = "Michael";

person1.sayName(); //"Nicholas"

sayNameForAll(); // "Michael" (the global variable is considered a property of the global object.)
```

#### Changing **this**

- call()

_(this, the rest of the parameters)_

```
sayNameForAll.call(this, "global"); // outputs "global:Michael"
sayNameForAll.call(person1, "person1"); // outputs "person1:Nicholas"
sayNameForAll.call(person2, "person2"); // outputs "person2:Greg"
```

Here the function is used as an object, not as a code to be execute.

- apply()

_(this, array or array-like object of parameters)_

Means you can use an arguments object as the second parameter.

```
sayNameForAll.apply(this, ["global"]); // outputs "global:Michael"
sayNameForAll.apply(person1, ["person1"]); // outputs "person1:Nicholas"
sayNameForAll.apply(person2, ["person2"]); // outputs "person2:Greg"
```

- bind()

_(this, all other arguments representing named parameters that should be permanently set in the new function)_

You can still pass in any parameters that aren’t permanently set later.

This means you can call a function without passing in any additional arguments, also, when a function is bound the value of this doesn’t change even if the same function is assgined to another object with different set of properties.

---

## Going hard-core with objects

Class-based languages lock down objects based on a class definition, JavaScript objects have no such restrictions.

When a property is first added to an object, JavaScript uses an internal method called **[[Put]]** on the object. The **[[Put]]** method creates a spot in the object to store the property

**This operation specifies not just the initial value, but also some attributes of the property. The result of calling [[Put]] is the creation of an own property on the object.**

When a new value is assigned to an existing property, a ­separate operation called **[[Set]]** takes place.

For checking if an object has a specific property regalrdless of if the value of that property is falsy:

```
"name" in person1 // true or false
```

The in operator checks for both own properties and prototype propertie, so we use hasOwnProperty() to check for the existance of properties that are in the object and not in the prototype:

```
console.log(person1.hasOwnProperty("name")); // true
console.log(person1.hasOwnProperty("toString")); // false
```

The delete operator works on a single object property and calls an internal operation named **[[Delete]]**.

```
console.log("name" in person1); // true
delete person1.name; // true - not output
console.log(person1.name); // false
```

Attempting to access a property that doesn’t exist will just return ndefined.

### Enumeration

By default, all properties that you add to an object are enumerable, which means that you can iterate over them using a for-in loop. Enumerable properties have their internal **[[Enumerable]]** attributes set to true .

We use **Object.keys()** to retreive an array of enumerable property names.

```
let properties = Object.keys(object);
```

**The for-in loop also enumerates prototype properties, while Object.keys() returns only own (instance) properties.**

To check whether a property is enumerable by using the **propertyIsEnumerable()** method:

```
console.log(person1.propertyIsEnumerable("name")); // true
```

Many native properties are not enumerable by default.

### Types of properties

Two different types of properties: data properties and accessor properties. Data properties contain a value, like the name property from earlier. The default behavior of the [[Put]] method is to create a data property.

**Accessor properties don’t contain a value but instead define a function to call _when the property is read_ (called a getter), and a function to call _when the property is written to_ (called a setter). Accessor properties only require either a getter or a setter, though they can have both.**

Using object literal:

```
let person1 = {
_name: "Nicholas",
get name() {
console.log("Reading name");
return this._name;
},
set name(value) {
console.log("Setting name to %s", value);
this._name = value;
}
};

console.log(person1.name); // "Reading name" then "Nicholas"
person1.name = "Greg";
console.log(person1.name); // "Setting name to Greg" then "Greg"
```

name here is an accessor property, while \_name is the data property.

(The leading underscore is a common convention to indicate that the property is considered to be private, though in reality it is still public.)

Even though this example uses \_name to store the property data, you could just as easily store the data in a variable or even in another object. This example simply adds logging to the behavior of the property; there’s usually no reason to use accessor properties if you are only storing the data in another property—just use the property itself. Accessor properties are most useful when you want the assignment of a value to trigger some sort of behavior, or when reading a value requires the calculation of the desired return value.

### Property attributes

- [[Enumerable]]: determines whether you can iterate over the property.
- [[Configurable]]: determines whether the property can be changed.

By default, all properties you declare on an object are both enumerable and configurable.

To change property attributes, we use the **Object.defineProperty()** method.**accepts three arguments: the object that owns the property, the property name, and a property descriptor object containing the attributes to set.**

```
let person1 = {
name: "Nicholas"
};

Object.defineProperty(person1, "name", {
enumerable: false
});

console.log("name" in person1); // true
console.log(person1.propertyIsEnumerable("name")); // false
let properties = Object.keys(person1);
console.log(properties.length); // 0

Object.defineProperty(person1, "name", {
configurable: false
});

// try to delete the Property
delete person1.name;
console.log("name" in person1); // true
console.log(person1.name); // "Nicholas"

Object.defineProperty(person1, "name", { // error
configurable: true
});
```

#### Data property attributes

- [[Value]]: holds the property value (All prop-erty values are stored in [[Value]] , even if the value is a function).
- [[Writable]]: a Boolean value indicating whether the property can be written to.

With these two additional attributes, you can fully define a data property using Object.defineProperty() even if the property doesn’t already exist.

This code

```
let person1 = {
name: "Nicholas"
};
```

is equivalent to:

```
let person1 = {};

Object.defineProperty(person1, "name", {
value: "Nicholas",
enumerable: true,
configurable: true,
writable: true
});
```

When Object.defineProperty() is called, it first checks to see if the property exists. If the property doesn’t exist, a new one is added with the attributes specified in the descriptor.

Boolean attributes automatically default to false, so when defining properties this way, make sure to set all the attributes.

#### Accessor property attributes

- [[Get]]: contians the getter function.
- [[Set]]: contains the setter function.

```
let person1 = {
_name: "Nicholas"
};

Object.defineProperty(person1, "name", {
get: function() {
console.log("Reading name");
return this._name;
},

set: function(value) {
console.log("Setting name to %s", value);
this._name = value;
},

enumerable: true,
configurable: true
});
```

We use **Object.defineProperties()** to define multiple properties at once.

### Retrieving property attributes

Using **Object.getOwnPropertyDescriptor()**, which works only on own properties, accepts two arguments: the object to work on and the property name to retrieve.

```
let descriptor = Object.getOwnPropertyDescriptor(person1, "name");

console.log(descriptor.enumerable);
console.log(descriptor.configurable);
console.log(descriptor.writable);
console.log(descriptor.value);
// they all return true or false except for the value attribute
```

### Preventing object modification

**Preventing Extensions**

Objects, just like properties, have internal attributes that govern their behavior.

- [[Extensible]]: a Boolean value indicating if the object itself can be modified, if false you can prevent new properties from being added to an object.

We use **Object.preventExtensions()**, and **Object.isExtensible()** to check.

```
console.log(Object.isExtensible(person1)); // true

Object.preventExtensions(person1);
console.log(Object.isExtensible(person1)); // false

person1.sayName = function() {
console.log(this.name);
};
console.log("sayName" in person1); // false
```

**Sealing Objects**

A sealed object is nonextensible, and all of its properties are nonconfigurable. That means not only can you not add new properties to the object,but you also can’t remove properties or change their type (from data to accessor or vice versa). If an object is sealed, you can only read from and write to its properties.

We use **Object.seal()** to seal an object, and **Object.isSealed()** to check.

**Freezing Objects**

A frozen object is a sealed object where data properties are also read-only.Frozen objects can’t become ­unfrozen.We use **Object.freeze()** and **Object.isFrozen()**.
