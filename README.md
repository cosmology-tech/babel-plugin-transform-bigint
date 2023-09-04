# babel-plugin-transform-bigint

*Update:* Now it can convert a code using BigInt into a code using JSBI (https://github.com/GoogleChromeLabs/jsbi).
It will try to detect when an operator is used for bigints, not numbers. This will not work in many cases, so please use JSBI directly only
if you know, that the code works only with bigints.

*Update by Cosmology tech:* Added config for changing jsbi lib string inside "import JSBI from 'jsbi';". More detail please see config section.

An example from https://github.com/GoogleChromeLabs/babel-plugin-transform-jsbi-to-bigint:
==========================================================================================

Input using native `BigInt`s:

```javascript
const a = BigInt(Number.MAX_SAFE_INTEGER);
const b = 2n;

a + b;
a - b;
a * b;
a / b;
a % b;
a ** b;
a << b;
a >> b;
a & b;
a | b;
a ^ b;

-a;
~a;

a === b;
a < b;
a <= b;
a > b;
a >= b;

a.toString();
Number(a);
```

Compiled output using `JSBI`:

```
const a = JSBI.BigInt(Number.MAX_SAFE_INTEGER);
const b = JSBI.BigInt(2);
JSBI.add(a, b);
JSBI.subtract(a, b);
JSBI.multiply(a, b);
JSBI.divide(a, b);
JSBI.remainder(a, b);
JSBI.exponentiate(a, b);
JSBI.leftShift(a, b);
JSBI.signedRightShift(a, b);
JSBI.bitwiseAnd(a, b);
JSBI.bitwiseOr(a, b);
JSBI.bitwiseXor(a, b);
JSBI.unaryMinus(a);
JSBI.bitwiseNot(a);
JSBI.equal(a, b);
JSBI.lessThan(a, b);
JSBI.lessThanOrEqual(a, b);
JSBI.greaterThan(a, b);
JSBI.greaterThanOrEqual(a, b);
a.toString();
JSBI.toNumber(a);
```

Config:
========

Users can change jsbi lib string in "import JSBI from './jsbi.mjs';" after transpiling by config in .babelrc.json:

e.g.:
```
  // this will change import into: import JSBI from 'jsbi';
  "plugins": [["@cosmology/babel-plugin-transform-bigint", { "jsbiLib": "jsbi" }]]
```

Example:
========

1. Create a folder "example".
2. Create a file test.js:
```javascript
// floor(log2(n)), n >= 1
function ilog2(n) {
  let i = 0n;
  while (n >= 2n**(2n**i)) {
    i += 1n;
  }
  let e = 0n;
  let t = 1n;
  while (i >= 0n) {
    let b = 2n**i;
    if (n >= t * 2n**b) {
      t *= 2n**b;
      e += b;
    }
    i -= 1n;
  }
  return e;
}

// floor(sqrt(S)), S >= 0, https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method
function sqrt(S) {
  let e = ilog2(S);
  if (e < 2n) {
    return 1n;
  }
  let f = e / 4n + 1n;
  let x = (sqrt(S / 2n**(f * 2n)) + 1n) * 2n**f;
  let xprev = x + 1n;
  while (xprev > x) {
    xprev = x;
    x = (x + S / x) / 2n;
  }
  return xprev;
}

function squareRoot(value, decimalDigits) {
  return (sqrt(BigInt(value) * 10n**(BigInt(decimalDigits) * 2n + 2n)) + 5n).toString();
}


```

3. Use babel:
```sh
npm init
npm install --save https://github.com/Yaffle/babel-plugin-transform-bigint
npm install --save-dev @babel/core @babel/cli
npm install jsbi --save
npm install --global http-server
npx babel --plugins=babel-plugin-transform-bigint test.js > test-transformed.js
http-server -p 8081
```

4. Comment out the next line in test-transformed.js:
```javascript
import JSBI from "./jsbi.mjs";
```

5. Create a file `test.html`.
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <script src="./node_modules/jsbi/dist/jsbi-umd.js"></script>
  <script src="test-transformed.js"></script>
  <script>
    document.addEventListener("DOMContentLoaded", function (event) {
      const form = document.querySelector("form");
      form.oninput = function () {
        const value = form.value.value;
        const digits = Number(form.digits.value);
        const s = squareRoot(value, digits);
        // IE 11 does not support <output> element, so `form.output.value = "value";` is not working.
        form.querySelector("output").textContent = '√' + value +  ' ≈ ' + s.slice(0, 0 - digits - 1) + '.' + s.slice(0 - digits - 1, -1) + '…';
      };
      form.oninput();
    }, false);
  </script>
  <style>
    output {
      overflow-wrap: break-word;
    }
  </style>
</head>
<body>
  <form>
    <div>
      <label for="value">Value:</label>
      <input id="value" name="value" type="number" min="2" step="1" value="2" />
    </div>
    <div>
      <label for="digits">Number of decimal digits:</label>
      <input id="digits" name="digits" type="number" min="1" step="1" value="100" />
    </div>
    <div>
      <output id="output" name="output" for="value digits" tabindex="0"></output>
    </div>
  </form>
</body>
</html>
```

6. Open `http://127.0.0.1:8081/test.html` in a web browser.
