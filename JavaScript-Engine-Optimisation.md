# JavaScript Engine Optimisation; A Reason For Types.

In this article I will look at a simple way to help optimise for JavaScript engine performance. To be clear I will not be looking at performance issues such latency, DOM manipulation, rendering, network latency, etc. These are all super important topics that warrant separate articles.

When you run JavaScript it is compiled by a JavaScript Engine. Traditionally, JavaScript code was translated to machine code using an interpreter. However, Javascript engines have evolved over the last two decades to run large and complex frameworks, to servers such as Node.js.

There are a few different JavaScript engines such as SpiderMonkey (Mozilla/Firefox), Chakra (Microsoft/Edge), V8 (Chrome, Node.js, Brave), JavaScriptCore (Safari). For IoT devices where memory is quite important, there are Javascript Engines such as JerryScript and DukTape.

Similarly to JavaScript, its engines follow the ECMAScript standard to implement and run your JavaScript code. This standard is governed by the TC39 Committee.

## Just In Time compilation (JIT)

Modern JavaScript engines have at least two compilers, one of them being an optimising compiler.

Instead of compiling ahead of time and then running the code, modern JavaScript Engines mixes these two steps together. The engine runs the source code, compiles it just when it’s needed, collects information, and then uses this information to recompile the code.

Different engines do this in slightly different ways, but the basic concept is the same; JavaScriptCore for example has three compilers, two of which are optimisers.

This kind of compilation brings with it better optimisation and different ways to increases the performance of your code.

One such way is in marking a function as "hot".

When the code is run, the compiler collects information about it such as how frequently it runs and the types that are being used. If the same lines of code are run a few times it will consider this code "warm". If the code is run a lot it will considered it "hot".

The optimising compiler uses the information it has gathered from running the function to speed up the process when it comes across this function again. It compiles assuming that it will see similar types as before to create optimised machine code.

The steps for this are:

-   Compile,
-   Run a few times,
-   Optimise assuming certain conditions (i.e. types are the same),
-   Run optimised code (machine code),
-   If conditions change fallback to default behaviour.

## Dynamically Typed

JavaScript however, is a dynamically typed language which means that variables can be created without specifying a type. Moreover, objects and variables can even be changed later to a totally different type.

```js
let example = 1; //no type declared; interpreted as type int  
example = "Hello 1!" // now interpreted as type string
```

Though this makes the language easier to program in, it also makes it harder for the engine to optimise and run your code. This is because there is no guarantee a function will receive the same types and therefore behave in the same way next time it runs.

Take this simple function for example:
```js
function find(obj) {  
   return obj.x;  
}
```
Though it’s only 3 lines of code it is fairly complex for the JavaScript engine. It has to ask:

-   does x exist?
-   is x a property of obj
-   is x a property of obj.prototype
-   if not, is x a part of the prototype chain?
-   where in memory is x?

The compiler has to do a great deal of lookup just to find the x property. In addition, the order of the properties also make a difference to the compiler. Just because two objects have the same properties, it doesn’t mean that they are represented as the same type internally.
```js
load({x: 1, y:2});  
load({x: 3, y:4});  
load({x: 5, y:6});  
load({x: 7, y:8});  
//Can be optimised: object properties are all of the same type and structure.
```
If for example you do not pass a property (in this case **_y_**) the engine will de-optimise this function as the object type is no longer consistent and is not represented as the same type internally.

```js 
load({x: 1,}); //function is de-optimised
```

To help the JavaScript compiler optimise your code it’s good practice to always use the same type of object. For example, if your objects represent the same thing (and it doesn’t make your code unreadable), you can make the engine know it’s the same type.
```js
load({x: 1, y:2});  
load({x: 3, y: undefined}); //Can be optimised: y is passed undefined instead of omission
```
This way the compiler will consider this as one object type, optimise the code, and later only have to do one comparison check.

If the code is optimised but types later change, there will be a performance drop when the engine has to de-optimise the code back to its default behaviour. This process is called de-optimisation or bailing out. It also ends up being slower than just executing the baseline compiled version of your code.

Most modern engines have added limits to break out of these optimisation/de-optimisation cycles when they happen. After a certain number of cycles it will stop trying to optimise the code.

## Conclusion

Modern JavaScript engines run faster by monitoring the code as it’s running it and making certain assumptions. It then uses the information gathered to optimise the code, assuming it will see similar types as before. This can result in many-fold performance improvements for most Javascript applications.

Javascript is a dynamically typed language which makes this process hard to guarantee. The engine has to keep in memory both versions of the code (optimised, default) and de-optimise to the default code if types change. This causes a performance hit each time, but to avoid this, the engine will only try to optimise the code a certain number of times.

Writing javascript code that is statically types in nature and consistent in its execution will help the compiler to optimise your code to faster machine code.