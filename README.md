# The Object Hotel
> Are objects as arguments pass by reference or pass by value? And why should we care?

I'll begin with the answer: JavaScript arguments are *always* passed by value, but for objects, the value of the variable *is* a reference. If you're new to JavaScript or to programming, it's not a very illuminating answer. Am I just being pedantic? I don't think so. "Pass a reference" and "pass by reference" are two very different things, as we will see.

When we dive into what it means for the value of a variable to be a reference, we'll see that the rules for variable assignment (and consequently argument passing) are actually completely consistent whether the variable contains a number, string, or object. And that's a really good thing: it lets us keep a very simple mental model when it comes to variable assignment, an operation so fundamental that it would be tragic if we had to scratch our heads and think about what it was doing every time we did it.

## Let's talk about variables

Variables are the simplest form of data storage available to us as programmers. A variable is like a container: you can put something in it, and then reference it later.

```javascript
var awesome = 1337;
```

"Putting something in" the variable is a well-defined process: the right-hand-side (RHS) of the assignment expression is evaluated, and the resulting single value is copied into the variable. This is always the process, regardless of what the RHS may consist of. Any expression, no matter
how complex, will always evaluate to one single value, which is all that a variable can hold.

It's also worth noting that assignment is the *only* way you can change the value of a variable, that is to say, they can only be modified directly. Conversely, changing the value of one variable will never affect the value of another variable. It's actually impossible to create a reference to a variable in JavaScript, since all we can do is copy values on assignment. Gotta use a lower level language like C to get that kind of access to memory.

#### Womp womp (JavaScript)
```javascript
var a = 'hello';
var b = a;
b = 'goodbye'; // a is not affected, since b only contained a copy of its value
```

#### Creating a reference (C)
```c
int  a = 0;   // variable a
int *p = &a;  // the memory address of a, that is, a real reference to a
    *p = 1;   // we remotely changed a's value by directly accessing its memory address!
```

C is pretty arcane, but a fun language to learn if you're interested at all in how nearly every other language actually works (most are implemented in C or C++). For now, we'll just borrow the concept of a "reference" and ignore all that wacky syntax.

## What does `var` do anyway?

Time to get a little technical. Consider the keyword `var`: ever wonder what it actually does? Sure, we know it creates a local variable in the current function scope, but that's an outcome not an action. Essentially, `var` allocates a slot of memory big enough to hold exactly one variable. Since JavaScript isn't typed, it's not exactly straightforward (a number might take up a different amount of space than a string), and reassignment may cause it to start all over again, but what's consistent here is that we're saying, "I have this single variable, with its single value and I need some space to store it."

The premise that I've insisted on so adamantly, that a variable only ever holds one value, appears to be completely contradicted when we throw objects into the mix. An object might contain any number of values as its properties, and we can definitely assign objects to variables.

```javascript
var myObj = {
    a: 1,
    b: 2,
    c: 3
};
```

Something's got to give: either it's time to stop reading this blog post, as the author, yours truly, is *clearly* an idiot, because I mean look, we just shoved three values into that variable no problem, or something else is going on. And in the face of this damning evidence, I *insist* that a variable can only hold one value, that is to say, you can't fit a whole object into a variable. There's just not enough space to store all its properties. So where do we put them?

## The Object Hotel

When an object is created, it checks in to what I like to call The Object Hotel. It goes up to the front desk and asks for a room. The concierge says "Oui monsieur, your room number is 0x12bcedfa." The object heads up the elevator, unpacks all of its stuff, kicks off its shoes, and hangs out in its room until it is forgotten about (just like in the Pixar movie [Coco](https://www.youtube.com/watch?v=RVNE60SFPpw))[.](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management#Garbage_collection)

What we store in the variable `myObj` is its room number.

## Beautiful consistency

With this model of objects in mind, all of the "strangeness" of objects and variables completely evaporates. Let's take a look at some examples.

```javascript
var a = {};
var b = {};

a === b; // false
```

Though the two objects are identical, `===` compares the two variable's *values*, and since they are distinct objects, they'll have different room numbers, and hence different values. The only time `===` will return true for two objects is if they have the same value: that is, they refer to the exact same object.

```javascript
var a = {};
var b = a;

b.what = 'huh?';

a.what; // 'huh?'
```

We copied `a`'s room number into `b` when we did the assignment. So when we dump stuff into `b`, assigning a new property, it ends up in the same room as the one referred to by a. When we look into `a`, it's no surprise that we see what we dumped in there via `b`, as they refer to the same physical location.

## Let's talk about function arguments

We're getting close to our answer now, but for completeness' sake, let's discuss what happens when we pass arguments to a function. Function parameters are basically variables: they contain a value, and when we pass in an argument, the value of the argument is copied into the parameter. Sounds an awful lot like regular assignment to me.

```javascript
var a = 0;
var b = 1;

function f (param1, param2) {
    // ...
}

f(a, b); // param1 gets mapped to a, param2 gets mapped to b
```

This is almost identical to saying something along the lines of

```
var a = 0;
var b = 1;

function f () {
    var param1 = a;
    var param2 = b;
    // ...
}
```

### What's "pass by reference"?

Pass by reference refers to passing an actual reference to the variable. In our C example, if we passed the pointer variable `p` to another function, the other function would be able to assign new values to `a`. In JavaScript, there's no such concept, and arguments are passed via assignment: consistently pass by value.

```javascript
var myObj = { key: 'value' };

function reassign (obj) {
    obj = { key: 'my new object!' };
}

reassign(myObj);
```

If objects were actually passed by reference, reassigning the `obj` parameter would cause the `myObj` variable to have changed. But it's not. Consistent with numbers, strings, and everything else, passing an object simply copies the value. What's different about objects, is that the value is a reference, and so inside the function, we are referring to the same object, the same room number in The Object Hotel.

```javascript
var myObj = { key: 'old value' };

function modify (obj) {
    obj.key = 'new value';
}

modify(myObj);

myObj.key; // it's 'new value' now
```

As such, we are able to modify objects passed into a function, in exactly the same way that we can when we assign a variable to an existing object. Nothing mysterious happens when we pass an object as an argument, it's just simple assignment.

#### What if I don't want my object to change?

To "safely" pass an object, we'll have to make a copy of it before passing it into the function (alternatively, we can make a copy of it inside the function). Once upon a time, to copy an object you had to loop over all of its properties and copy them manually, but in 2018 we have some pretty nifty ways to copy an object.

 One way is to use the ES6 function `Object.assign`. This takes a target object, and then copies all the properties of the rest of the arguments to the target. Very useful for mashing objects together, and for our purposes, makes copying an object a simple one-liner. We call it with an empty object, and the object we want to copy.

```javascript
var myObj = { key: 'value' };
var copy = Object.assign({}, myObj);
```

Now we can pass the copied object to any function, and `myObj` will remain safe and sound.

## Arrays and Functions are objects too

Everything about the variables being stored as references applies to arrays and functions as well. If you pass an array to a function, it's liable to be modified, and have those modifications reflect on the outside. Same with functions, though we usually leave those pretty much alone.

## Ok but a hotel, seriously?

Actually, minus the stereotypical French concierge, this is basically how it works under the hood. If we take our metaphor and replace the word "hotel" with "heap" and instead of room numbers, call them "memory addresses" then it becomes a decent description of how the memory allocation for objects works. But it's more fun and memorable when we personify the objects and give them a nice 5-star hotel to relax in, I think.

So that's that: objects are special in that they are references, but variables themselves behave in a simple and consistent way, always copying the right hand side into the left hand side. I think drawing the distinction in the right place is important, so that our mental models can be as simple and consistent as possible.

#### Thanks for reading! Since this is just a GitHub repo, you can open an issue to "leave a comment". (Or make a pull request if I've gotten something *horribly wrong*)
