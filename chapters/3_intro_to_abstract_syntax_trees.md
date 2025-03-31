# Introduction to Abstract Syntax Trees (AST)
An Abstract Syntax Tree is a hierarchical representation of the structure of some source code. 
It will break down code into a tree of nodes, each of which representing a syntactic construct like variables, functions, loops, or operators. 
ASTs are foundational to compilers, linters, and general code transformation tools; as a result they are incredibly powerful for reverse engineering web applications.

## Why ASTs Matter in Web Reverse Engineering:
- We can deobfuscate code by renaming variables and simplifying expressions.
- We can reverse logic by reconstructing control flow.

## Generating an AST
```js
function greet(name) {
  return "Hello " + name + "!";
}
```

Using a tool like [AST Explorer](https://astexplorer.net/), we can generate its AST:
```json
{
  "type": "Program",
  "start": 0,
  "end": 56,
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 56,
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 14,
        "name": "greet"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [
        {
          "type": "Identifier",
          "start": 15,
          "end": 19,
          "name": "name"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "start": 21,
        "end": 56,
        "body": [
          {
            "type": "ReturnStatement",
            "start": 25,
            "end": 54,
            "argument": {
              "type": "BinaryExpression",
              "start": 32,
              "end": 53,
              "left": {
                "type": "BinaryExpression",
                "start": 32,
                "end": 47,
                "left": {
                  "type": "Literal",
                  "start": 32,
                  "end": 40,
                  "value": "Hello ",
                  "raw": "\"Hello \""
                },
                "operator": "+",
                "right": {
                  "type": "Identifier",
                  "start": 43,
                  "end": 47,
                  "name": "name"
                }
              },
              "operator": "+",
              "right": {
                "type": "Literal",
                "start": 50,
                "end": 53,
                "value": "!",
                "raw": "\"!\""
              }
            }
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```

**1. Observations:**
- Nodes like FunctionDeclaration and Identifier map directly to code constructs.
- Nesting of statements is perserved in the tree structure (e.g ReturnStatement inside BlockStatement).

**2. Traversing and Modifying**

Tools like Babel allow programmatic traversal and modification of ASTs. 

With Bable we can use visitor patterns to target specific node types.

e.g Rename all variables named x to secretVar:

```js
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const generator = require("@babel/generator").default;

const code = "let x = 42; console.log(x);";
const ast = parser.parse(code);

traverse(ast, {
  Identifier(path) {
    if (path.node.name === "x") {
      path.node.name = "secretVar";
    }
  },
});

console.log(generator(ast).code);
```
Output: 
> let secretVar = 42; console.log(secretVar);

## Deobfuscating String Concatenation
Obfuscated:
```js
const s = ["He", "llo"].join("") + " World";
```

**Approaching deobfucsating this through AST parsing**:
1. Identify BinaryExpression and CallExpression nodes.
2. Evaluate static string operations (e.g ["He", "llo"].join("") -> "Hello").
3. Replace the original nodes with your new simplified literal nodes.

Deobfuscated:
```js
const s = "Hello World";
```

## Tools for AST Manipulation
- [AST Explorer](https://astexplorer.net/): Web-based tool for visualizing and experimenting with ASTs.
- [Babel](https://github.com/babel/babel): Parse, traverse, and generate code.
- [Recast](https://github.com/benjamn/recast): AST-to-code conversion with formatting preservation.

## Challenge
TODO: // AST challenge

--- 

<p align="center">
  <a href="/web-reversing/chapters/2_understanding_javascript_quirks">← Previous Chapter</a> |
  <a href="/web-reversing/chapters/4_intro_to_transformation_obfuscation">Next Chapter →</a>
</p>