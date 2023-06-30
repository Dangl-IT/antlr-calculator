﻿# antlr-calculator

[![Build Status](https://jenkins.dangl.me/buildStatus/icon?job=GeorgDangl%2Fantlr-calculator%2Fdevelop)](https://jenkins.dangl.me/job/GeorgDangl/job/antlr-calculator/job/develop/)
[![npm](https://img.shields.io/npm/v/antlr-calculator.svg)](https://www.npmjs.com/package/antlr-calculator)

This calculator is using the [ANTLR4 TypeScript target](https://github.com/tunnelvisionlabs/antlr4ts)
to calculate results from formulas that are passed in as string. Both JavaScript and TypeScript are supported.

Whenever a calculation is performed, a `CalculationResult` is returned with the following properties:

| Property      | Type    |                                                                                             |
| ------------- | ------- | ------------------------------------------------------------------------------------------- |
| isValid       | boolean | `true` if the formula could be parsed and calculated, else `false`                          |
| errorPosition | number  | Position of the offending symbol in the line, 0 based index, for invalid results, else null |
| errorMessage  | string  | ANTLR error message for invalid formulas, else null                                         |
| result        | number  | `NaN` for invalid formulas, else the actual result                                          |

[You can check out the live demo here!](https://antlr-calculator.dangl.me)

You can find the .NET version here: https://github.com/GeorgDangl/Dangl.Calculator

## Installation

Clone this repository or just go with `npm install antlr-calculator`.

## JavaScript

Just reference `dist/bundle.js` and the global variable `antlrCalc` is available.

```javascript
var result = antlrCalc.Calculator.calculate('4*5');
console.log(JSON.stringify(result, null, 2));

//  {
//    "isValid": true,
//    "errorPosition": null,
//    "errorMessage": null,
//    "result": 20
//  }
```

## TypeScript

Import the `Calculator` class and use the static `calculate(formula: string)` method to evaluate formulas.

```typescript
import { Calculator } from 'antlr-calculator';

var result = Calculator.calculate('4*5');
console.log(JSON.stringify(result, null, 2));

//  {
//    "isValid": true,
//    "errorPosition": null,
//    "errorMessage": null,
//    "result": 20
//  }
```

## Supported functions

| Expression                                   |                                                                 |
| -------------------------------------------- | --------------------------------------------------------------- |
| `FLOOR expression`                           | Round down to zero accuracy                                     |
| `CEIL expression`                            | Round up to zero accuracy                                       |
| `ABS expression`                             | Absolute value                                                  |
| `ROUNDK '(' expression ';' expression ')'`   | Round expr_1 with expr_2 accuracy                               |
| `ROUND expression`                           | Round with zero accuracy                                        |
| `TRUNC expression`                           | Trim decimal digits                                             |
| `SIN expression`                             | Sinus                                                           |
| `COS expression`                             | Cosinus                                                         |
| `TAN expression`                             | Tangens                                                         |
| `COT expression`                             | Cotangens                                                       |
| `SINH expression`                            | Sinus Hypererbolicus                                            |
| `COSH expression`                            | Cosinus Hyperbolicus                                            |
| `TANH expression`                            | Tangens Hyperbolicus                                            |
| `ARCSIN expression`                          | Inverse Sinus                                                   |
| `ARCCOS expression`                          | Inverse Cosinus                                                 |
| `ARCTAN expression`                          | Inverse Tangens                                                 |
| `ARCTAN2 '(' expression ';' expression ')'`  | Atan2                                                           |
| `ARCCOT expression`                          | Inverse Cotangens                                               |
| `EXP expression`                             | e ^ expr                                                        |
| `LN expression`                              | Logarithm to e                                                  |
| `EEX expression`                             | 10 ^ expr                                                       |
| `LOG expression`                             | Logarithm to 10                                                 |
| `RAD expression`                             | Angle to radians (360° base)                                    |
| `DEG expression`                             | Radians to angle (360° base)                                    |
| `SQRT expression`                            | Square root                                                     |
| `SQR expression`                             | Square product                                                  |
| `expression op = ('^' \| '\*\*') expression` | expr_1 to the expr_2 th power                                   |
| `expression (MOD \| '%' ) expression`        | Modulo                                                          |
| `expression DIV expression`                  | Whole part of division rest                                     |
| `expression op = ('~' \| '//') expression`   | expr_1 nth root of expr_2                                       |
| `expression op = ('\*' \| '/') expression`   | Multiplication or division                                      |
| `expression op = ('+' \| '-') expression`    | Addition or subtraction                                         |
| `NUMBER`                                     | Single integer or float number                                  |
| `'(' expression ')'`                         | Expression within parentheses                                   |
| `MIN '(' expression (';' expression)* ')'`   | Minimum                                                         |
| `MAX '(' expression (';' expression)* ')'`   | Maximum                                                         |
| `PI '()'?`                                   | Mathematical constant pi = 3,141593                             |
| `expression E+ expression`                   | Exponent, e.g. 10e+43                                           |
| `expression E- expression`                   | Inverted Exponent, e.g. 10e-43                                  |
| `EULER`                                      | Mathematical constant e = 2,718282                              |
| `'-' expression`                             | Unary minus sign (negative numbers)                             |
| `'+' expression`                             | Unary plus sign (positive numbers)                              |
| `'(' expression ')' expression`              | Expressions without multiplication sign, e.g. `2(3)` -> `2*(3)` |
| `expression '(' expression ')'`              | Expressions without multiplication sign, e.g. `2(3)` -> `2*(3)` |

_expression_ may be any expression as functions can be nested. Example: `DEG(2*PI)` or `LOG(10^3)`.

Formulas can be case invariant, e.g. `SIN`, `sin` and `siN` are all considered the same.

## Comments in Formulas

Comments in Formulas are supported by encapsulating them either in `/*...*/`, `'...'` or `"..."` quote styles. Examples:

`4/*Length*/*3/*Width*/` resolves to `12`

`4'Length'*3'Width'` resolves to `12`

`4"Length"*3"Width"` resolves to `12`

## Substitutions

The calculator can be called with an overload that accepts a callback function for substitution values. For example, take the following formula:  
`1,2*#Z4+3`  
Here, `#Z4` is a _substitution_, which is a placeholder that can be externally supplied. Let's say you want to resolve `#Z4` to the value three, you could make this simple call:

```typescript
const formula = '1,2*#Z4+3';
const result = Calculator.calculate(formula, (substitution) => {
  if (substitution === '#Z4') {
    return 3;
  }

  return null;
});
```

The callback is in the form of a `(substitution: string) => number`, and it will be called for every substitution found in the formula. Multiple substitutions are supported. If duplicates in substitutions are present, the calculator will request each one individually. If a substitution resolves to `null`, the formula is considered invalid.

Substitutions must always start with the `#` character and can then have the following characters: `[a-z] | [A-Z] | [äÄöÖüÜ] | [0-9]`

## Trailing comments

Formulas may be terminated with a semicolon `;` at the end, followed by extra input that is not evaluated. This is useful when, instead of regular comments, you
just want to attach some trailing formation at the end of a formula. For example, the following formula:  
`1 + 3; As per our counting`  
Would just evaluate the `1 + 3` portion and return a valid result with the value `4`, ignoring the trailing semicolon and all input that follows.

---

[MIT License](License.md)
