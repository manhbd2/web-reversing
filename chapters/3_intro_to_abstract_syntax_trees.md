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
```json
{
  "type": "Program",
  "start": 0,
  "end": 313,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 67,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 8,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 8,
            "name": "WH4T"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 10,
          "end": 15,
          "id": {
            "type": "Identifier",
            "start": 10,
            "end": 15,
            "name": "C0ULD"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 17,
          "end": 24,
          "id": {
            "type": "Identifier",
            "start": 17,
            "end": 24,
            "name": "R3VERS3"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 26,
          "end": 37,
          "id": {
            "type": "Identifier",
            "start": 26,
            "end": 37,
            "name": "ENG1N33R1NG"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 39,
          "end": 47,
          "id": {
            "type": "Identifier",
            "start": 39,
            "end": 47,
            "name": "ABSTR4CT"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 49,
          "end": 55,
          "id": {
            "type": "Identifier",
            "start": 49,
            "end": 55,
            "name": "SYNT4X"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 57,
          "end": 62,
          "id": {
            "type": "Identifier",
            "start": 57,
            "end": 62,
            "name": "TR33S"
          },
          "init": null
        },
        {
          "type": "VariableDeclarator",
          "start": 64,
          "end": 66,
          "id": {
            "type": "Identifier",
            "start": 64,
            "end": 66,
            "name": "D0"
          },
          "init": null
        }
      ],
      "kind": "let"
    },
    {
      "type": "ExpressionStatement",
      "start": 69,
      "end": 313,
      "expression": {
        "type": "CallExpression",
        "start": 69,
        "end": 312,
        "callee": {
          "type": "FunctionExpression",
          "start": 70,
          "end": 309,
          "id": {
            "type": "Identifier",
            "start": 79,
            "end": 92,
            "name": "flagChallenge"
          },
          "expression": false,
          "generator": false,
          "async": false,
          "params": [],
          "body": {
            "type": "BlockStatement",
            "start": 95,
            "end": 309,
            "body": [
              {
                "type": "VariableDeclaration",
                "start": 101,
                "end": 144,
                "declarations": [
                  {
                    "type": "VariableDeclarator",
                    "start": 107,
                    "end": 143,
                    "id": {
                      "type": "Identifier",
                      "start": 107,
                      "end": 111,
                      "name": "data"
                    },
                    "init": {
                      "type": "BinaryExpression",
                      "start": 114,
                      "end": 143,
                      "left": {
                        "type": "BinaryExpression",
                        "start": 114,
                        "end": 137,
                        "left": {
                          "type": "BinaryExpression",
                          "start": 114,
                          "end": 130,
                          "left": {
                            "type": "Identifier",
                            "start": 114,
                            "end": 121,
                            "name": "R3VERS3"
                          },
                          "operator": "-",
                          "right": {
                            "type": "Identifier",
                            "start": 122,
                            "end": 130,
                            "name": "ABSTR4CT"
                          }
                        },
                        "operator": "-",
                        "right": {
                          "type": "Identifier",
                          "start": 131,
                          "end": 137,
                          "name": "SYNT4X"
                        }
                      },
                      "operator": "-",
                      "right": {
                        "type": "Identifier",
                        "start": 138,
                        "end": 143,
                        "name": "TR33S"
                      }
                    }
                  }
                ],
                "kind": "const"
              },
              {
                "type": "VariableDeclaration",
                "start": 149,
                "end": 186,
                "declarations": [
                  {
                    "type": "VariableDeclarator",
                    "start": 155,
                    "end": 185,
                    "id": {
                      "type": "Identifier",
                      "start": 155,
                      "end": 158,
                      "name": "src"
                    },
                    "init": {
                      "type": "CallExpression",
                      "start": 161,
                      "end": 185,
                      "callee": {
                        "type": "MemberExpression",
                        "start": 161,
                        "end": 183,
                        "object": {
                          "type": "Identifier",
                          "start": 161,
                          "end": 174,
                          "name": "flagChallenge"
                        },
                        "property": {
                          "type": "Identifier",
                          "start": 175,
                          "end": 183,
                          "name": "toString"
                        },
                        "computed": false,
                        "optional": false
                      },
                      "arguments": [],
                      "optional": false
                    }
                  }
                ],
                "kind": "const"
              },
              {
                "type": "VariableDeclaration",
                "start": 191,
                "end": 238,
                "declarations": [
                  {
                    "type": "VariableDeclarator",
                    "start": 197,
                    "end": 237,
                    "id": {
                      "type": "Identifier",
                      "start": 197,
                      "end": 202,
                      "name": "regex"
                    },
                    "init": {
                      "type": "NewExpression",
                      "start": 205,
                      "end": 237,
                      "callee": {
                        "type": "Identifier",
                        "start": 209,
                        "end": 215,
                        "name": "RegExp"
                      },
                      "arguments": [
                        {
                          "type": "Literal",
                          "start": 216,
                          "end": 236,
                          "value": "data\\s=\\s([^;]+)",
                          "raw": "\"data\\\\s=\\\\s([^;]+)\""
                        }
                      ]
                    }
                  }
                ],
                "kind": "const"
              },
              {
                "type": "ExpressionStatement",
                "start": 243,
                "end": 307,
                "expression": {
                  "type": "CallExpression",
                  "start": 243,
                  "end": 306,
                  "callee": {
                    "type": "MemberExpression",
                    "start": 243,
                    "end": 254,
                    "object": {
                      "type": "Identifier",
                      "start": 243,
                      "end": 250,
                      "name": "console"
                    },
                    "property": {
                      "type": "Identifier",
                      "start": 251,
                      "end": 254,
                      "name": "log"
                    },
                    "computed": false,
                    "optional": false
                  },
                  "arguments": [
                    {
                      "type": "TemplateLiteral",
                      "start": 255,
                      "end": 305,
                      "expressions": [
                        {
                          "type": "CallExpression",
                          "start": 263,
                          "end": 302,
                          "callee": {
                            "type": "MemberExpression",
                            "start": 263,
                            "end": 293,
                            "object": {
                              "type": "MemberExpression",
                              "start": 263,
                              "end": 282,
                              "object": {
                                "type": "CallExpression",
                                "start": 263,
                                "end": 279,
                                "callee": {
                                  "type": "MemberExpression",
                                  "start": 263,
                                  "end": 272,
                                  "object": {
                                    "type": "Identifier",
                                    "start": 263,
                                    "end": 266,
                                    "name": "src"
                                  },
                                  "property": {
                                    "type": "Identifier",
                                    "start": 267,
                                    "end": 272,
                                    "name": "match"
                                  },
                                  "computed": false,
                                  "optional": false
                                },
                                "arguments": [
                                  {
                                    "type": "Identifier",
                                    "start": 273,
                                    "end": 278,
                                    "name": "regex"
                                  }
                                ],
                                "optional": false
                              },
                              "property": {
                                "type": "Literal",
                                "start": 280,
                                "end": 281,
                                "value": 1,
                                "raw": "1"
                              },
                              "computed": true,
                              "optional": false
                            },
                            "property": {
                              "type": "Identifier",
                              "start": 283,
                              "end": 293,
                              "name": "replaceAll"
                            },
                            "computed": false,
                            "optional": false
                          },
                          "arguments": [
                            {
                              "type": "Literal",
                              "start": 294,
                              "end": 297,
                              "value": " ",
                              "raw": "\" \""
                            },
                            {
                              "type": "Literal",
                              "start": 299,
                              "end": 301,
                              "value": "",
                              "raw": "\"\""
                            }
                          ],
                          "optional": false
                        }
                      ],
                      "quasis": [
                        {
                          "type": "TemplateElement",
                          "start": 256,
                          "end": 261,
                          "value": {
                            "raw": "flag{",
                            "cooked": "flag{"
                          },
                          "tail": false
                        },
                        {
                          "type": "TemplateElement",
                          "start": 303,
                          "end": 304,
                          "value": {
                            "raw": "}",
                            "cooked": "}"
                          },
                          "tail": true
                        }
                      ]
                    }
                  ],
                  "optional": false
                }
              }
            ]
          }
        },
        "arguments": [],
        "optional": false
      }
    }
  ],
  "sourceType": "module"
}
```

--- 

<p align="center">
  <a href="/web-reversing/chapters/2_understanding_javascript_quirks">← Previous Chapter</a> |
  <a href="/web-reversing/chapters/4_intro_to_transformation_obfuscation">Next Chapter →</a>
</p>