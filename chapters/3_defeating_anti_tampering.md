# Defeating Anti-Tampering Mechanisms
Now that we have been introduced to transformation obfuscation, you may have noticed that its still possible to analyse and modify the code, defeating any transformations that have taken place.
Good obfuscators don't like it when we modify the code, lets discuss some ways they may try to prevent us from manipulating/analysing the code.

## Generic debugger traps
### 1. Debugger spam
```javascript
setInterval(() => { debugger }, 10);
```
One of the most common anti-tampering techniques and most easily defeated, since debuggers are only handled when the devtools thread is attached, this will do nothing to standard users.
This can be defeated by pressing <kbd>Ctrl</kbd> + <kbd>F8</kbd> or clicking the `Deactivate breakpoints` button in the sources tab.
Or just pre-patching the code with a debugger hook:
```js
window._originalDebugger = window.debugger;
window.debugger = () => {};
```

### 2. Timing checks
```javascript
const start = performance.now();
// Critical code block
console.log("Hello world! :D\nI sure hope no one puts a breakpoint on me for more than 100ms... :clueless:")
// Critical code block end
const end = performance.now();
if (end - start > 100) { 
  triggerAntiTamper();
}
```
Also incredibly common, we can defeat this through a few different attack vectors, one of the simplest approaches being hooking performance.now:
```js
window._originalPerfNow = performance.now;
window.performance.now = () => 0;
```
Keep in mind the performance API isnt the only way this can be done, but its a pretty obvious pattern once you learn to recognise it.

## Devtools detection via console API
```js
const err = new Error();
Object.defineProperty(err, 'stack', {
  get() {
    console.log("You're detection buddy... :(")
  }
});
console.log(err);
```
One of many possible ways to detect the console, overriding properties which will be automatically rendered by the console, this can be done with so much that I would just stick with detatching the console API entirely like we do in chapter `0`.
Since we're detatching it in `resource://devtools/server/actors/webconsole/listeners/console-api.js` the same way as having no handler when devtools are closed, this defeats all these detection vectors.

## Hashing expected data
This is one of the better ways to approach security/anti-tampering, its purely creative and there's no one way to defeat it.
For instance, in a JSVM we could dynamically generate then hash some instructions to manage control flow (Dont worry if you're unfamiliar with these concepts we'll discuss them in later chapters):
```js
case "specificInstruction":
  const hash = CryptoJS.SHA1(instruction).toString().slice(0, 8);
  switch(hash) {
    case "a94a8fe5": return routineA(instruction);
    case "d6d6d6d6": return routineB(instruction);
    default: throw new Error("Invalid control flow");
  }
```
You can even bruteforce segments of a secure hash to preimage segments of your own hash within your function.
```js
function preimagedHashFunc(){require("crypto").createHash("sha256").update(arguments.callee.toString()).digest("hex").startsWith("3")?console.log("Running super secure operation..."):console.log("Modification detected! >:(")}
preimagedHashFunc();
```

## Self-referential logic 
TODO: // Finish self referencing transformer

## Challenge
Slightly big file:
[Download challenge][1]
[1]:/web-reversing/download/anti-tampering-challenge.txt

# Clues to defeat (spoilers):

<details>
  <summary>Step 1</summary>
  
  Iterate all functions and index the `length` and the `.toString()`.`length` of each.
</details>

<details>
  <summary>Step 2</summary>
  
  Replace `func['length']` with the indexed length and `[func+[]][+[]]['length']` with the toStringed length
</details>

<details>
  <summary>Step 3</summary>
  
  Arbitrary modification is now possible, you can now inline all operation functions regardless of parameter count e.g 

  function mult(a, b, c, e) { return a * b } -> a * b
</details>

<details>
  <summary>Step 4</summary>
  
  Isolate the statements that direct to each branch.

  Either modify the statement or extract the specific charcodes which contain the flag.
</details>

---

<p align="center">
  <a href="/web-reversing/chapters/2_intro_to_transformation_obfuscation">← Previous Chapter</a> 
</p>