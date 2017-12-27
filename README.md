Search an object's code for pattern matches

### Synopsis

```js
function(pattern, options)
```

### Parameters
0. `this` (the execution context) is "the module," the object containing the code to search
1. `pattern` is the required search argument, a string or (more typically) a [`RegExp`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/RegExp) pattern
2. `options` is an optional options object (see below)

### Returns
A string array containing the found matches.

### Description
It is important to note that `code-match` can only search code in methods, getters, and setters.
Code hidden in unexposed "inner" module functions is also hidden from `code-match`.

Although `code-match` cannot search inner functions, it can do a recursive walk through the context object's member objects.
If you are aware of code included in such nested objects, set [`options.recurse`](#optionsrecurse) to truthy.
Note that circular references are excluded from recursion, as are all references to DOM nodes.

By default `code-match` searches a catalog that includes the execution context object _and its prototype chain._
To skip walking the prototype chain, set [`options.catalog.own`](#optionscatalogown) to truthy.

(Don't confuse match recursion with cataloging the prototype chain. These are two different walk axes.)

Mix `code-match` into an object on which it will be routinely called. The reference to itself will be ignored.

Users should consider blacklisting keys associated with other foreign mix-ins.
Simply include references to those mix-ins in [`options.greylist.black`](#optionsgreylistblack).

### Example

```js
var codeMatch = require('code-match');
var A = require('module'), B = require('submodule');

codeMatch.call(A, A.eventStringPattern); // ['click', 'keypress']
codeMatch.call(B, B.eventStringPattern); // ['keydown', 'keypress']

var options = { catalog: { own: true } }; // skip the prototype chain
codeMatch.call(B, B.eventStringPattern, options); // ['keydown']
```

##### module.js
A simple module containing a couple of methods and a search pattern definition:
```js
module.exports = {
    handleMouseClick: function() { document.addEventListener('click', function() {...}); },
    handleKeyPress: function() { document.addEventListener('keypress', function() {...}); },
    eventStringPattern: /\.addEventListener\('([a-z]+)'/;
};
```

##### submodule.js
Here the two methods have been moved to the "superclass" (immediate predecessor in the prototype chain),
with one of those methods being overridden by the descendant:
```js
module.exports = Object.create(require('module'));
module.exports.handleMouseClick = function() { document.addEventListener('keydown', function() {...}); };
```

### Options

#### `options.captureGroup`
Iff defined, index of a specific capture group to return for each match.

#### `options.recurse`
Equivalent to setting both `recurseOwn` and `recurseAncestors`.

#### `options.recurseOwn`
Recurse on own nested objects.

#### `options.recurseAncestors`
Recurse on prototype chain's nested objects.

#### `options.greylist.white`
A whitelist of permissible code matches.
Only listed matches are included in the results.
If `undefined`, all matches are included.
If an empty array, all matches are blocked.

#### `options.greylist.black`
A blacklist of impermissible code matches.
Listed object matches are excluded from the results.
If `undefined` or an empty array, all matches are included.

#### `options.catalog.own`
If truthy, the resultant catalog is restricted to the execution context object only (excluding the prototype chain).

#### `options.catalog.greylist.white`
A whitelist of permissible object member keys to catalog.
Only listed object members are cataloged.
If `undefined`, all members are cataloged.
If an empty array, all members are blocked.

#### `options.catalog.greylist.black`
A blacklist of impermissible object member keys.
Listed object members are blocked.
If `undefined` or an empty array, all members that passed the whitelist are included.

#### Regarding the `greylist` options

Each of the above `.greylist.white` or `.greylist.black` options may be:
* a string; or
* a regular expression; or
* an object (_i.e.,_ its enumerable defined keys); or
* a (nested) array of a mix of any of the above; or
* an empty array; or
* `undefined` (which is a no-op).

See [greylist](https://github.com/joneit/greylist) for more information.
