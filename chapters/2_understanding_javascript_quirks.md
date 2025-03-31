# Understanding JavaScript Quirks
JavaScript is powerful but certainly has its flaws. 
Its design includes features that can lead to unexpected behavior if not understood. 
Let's explore some of its quirks...

## Weakly Typed
JavaScript is dynamically and weakly typed, meaning variables can change types implicitly during operations.

Unlike strongly typed languages, JavaScript performs automatic type conversions (coercion) in expressions:
```js
let x = 10;    // Number
x = "hello";   // Now a string (no error)
```

This can lead to strange results...

## Weird Behavior from Type Inference
There is so much strange behaviour I could write an essay on this alone, I would encourage you to check out:
> https://github.com/denysdovhan/wtfjs

But lets cover some of the simple things.
JavaScript's type coercion rules determine how operators behave:
```js
// Number + Number -> Addition
console.log(2 + 3);      // 5

// Boolean + Number -> Addition (Boolean converted to 0/1)
console.log(true + 5);   // 6 (true -> 1)

// Boolean + Boolean -> Addition
console.log(true + false); // 1 (1 + 0)

// Number + String -> Concatenation
console.log(5 + "5");    // "55"

// String + Boolean -> Concatenation
console.log("Hi" + true); // "Hitrue"

// String + String -> Concatenation
console.log("a" + "b");  // "ab"
```

## Looking into JSF*ck
An real application of this behaviour is [JSF*ck](https://jsfuck.com/).
JSF\*ck is an esoteric subset of JavaScript that uses only six characters:
- `[`
- `]`
- `(`
- `)`
- `!`
- `+`

To write executable code, exploiting JavaScript's coercion rules:
- false -> ![] (since [] is truthy, ![] becomes false).
- 0 -> +[] (empty array converted to 0).


How letters can be extracted from coerced strings:
```js
// 'a' is at index 1 in "false":
console.log((![]+[])[+!+[]]); // "a"
```

How evaluation is handled via `constructors` in order to execute arbitrary code:
```js
(() => {}).constructor("console.log('Hello world!')")()
```
In JavaScript all functions have constructors, which are used to build functions; due to the "everything is an object" nature of JavaScript, we can access that property on every single function.

How does JSF*ck abuse this:
```js
[]["filter"]["constructor"]( CODE )()
```

Here JSF*ck is doing the following: 
- Creating a new array `[]`,
- Accessing the `filter` property (builtin function for arrays),
- Getting the constructor property of that function.
- Building its own function using that and calling it.

Since we can coerce arbitrary strings through type coercion, we can build functions from just the aformentioned 6 characters.
What this looks like:
```js
[][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()([+!+[]]+[])
```

## Simple attack vector
Overriding eval we can make the browser do all the reconstruction for us.
```js
function evalDecode(source) {
  let _eval = globalThis.eval;
  globalThis.eval = (code) => (globalThis.eval = _eval, code);
  return _eval(source);
}

const code = `[][(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()((![]+[])[+!+[]]+(!![]+[])[+!+[]]+([][(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(![]+[])[!+[]+!+[]+!+[]]]()+[])[!+[]+!+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+!+[]]+(+[![]]+[+(+!+[]+(!+[]+[])[!+[]+!+[]+!+[]]+[+!+[]]+[+[]]+[+[]]+[+[]])])[+!+[]+[+[]]]+(+[![]]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+([![]]+[][[]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(+(!+[]+!+[]+[+!+[]]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[+!+[]])[+!+[]]+([][[]]+[])[+[]]+(!![]+[])[+[]])`;
console.log(evalDecode(code));
```

## Other Strange Quirks
> Truthy but Not true

Values are truthy if they evaluate to true in a boolean context, but aren't necessarily equal to true:
```js
if ("0") { 
  console.log("Runs"); // "0" is truthy
}
console.log("0" == true); // false ("0" -> 0, true -> 1 -> 0 ≠ 1)
```

> Falsy but Not false

Falsy values include false, 0, "", null, undefined, and NaN. However, comparisons can be misleading:
```js
console.log([] == false); // true ([] -> "" -> 0, false -> 0)
console.log(!![]);        // true ([] is truthy)
```

## Challenge
```js
// I seem to have lost my flag inside this weird datastructure, can you extract the charcodes and re-assemble it?
let l = {
  [{}]: {}
};
Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
  l[{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
    l[{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
      l[{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
        l[{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
          l[{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
            l[{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
              l[{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                l[{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                  l[{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                    l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                      l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                        l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                          l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                            l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                              l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                                l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                                  l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                                    l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length, l["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"]["[object Object]"] = {}, Function.constructor.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.call.apply((function() {
                                      l[{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}][{}].k = arguments.callee.caller.toString().match(/constructor\.(.+)(?=apply)/)[0].split("call").length
                                    }))
                                  }))
                                }))
                              }))
                            }))
                          }))
                        }))
                      }))
                    }))
                  }))
                }))
              }))
            }))
          }))
        }))
      }))
    }))
  }))
})); 
console.log(l);
```

<p align="center"> 
  <a href="/web-reversing/chapters/1_understanding_packers">← Previous Chapter</a> | 
  <a href="/web-reversing/chapters/3_intro_to_abstract_syntax_trees">Next Chapter -></a> 
</p>