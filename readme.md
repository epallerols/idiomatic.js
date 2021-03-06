# Principles of Writing Consistent, Idiomatic JavaScript

All code in any code-base should look like a single person typed it, no matter how many people contributed.

## Table of Contents

 * [Whitespace](#whitespace)
 * [Syntax](#syntax)
 * [Type Checking (Courtesy jQuery Core Style Guidelines)](#type)
 * [Conditional Evaluation](#cond)
 * [Practical Style](#practical)
 * [Naming](#naming)
 * [Misc](#misc)
 * [Native & Host Objects](#native)
 * [Comments](#comments)
 * [One Language Code](#language)



------------------------------------------------


## Code styling

1. <a name="syntax">Syntax</a>

    A. Whitespaces

    - Use always tabs.
    - Use always double quotes.
    - Trim trailing white spaces.

    B. Parenthesis, Braces and Linebreaks

    `if`/`else`/`for`/`while`/`try` always have spaces, braces and span multiple lines this encourages readability.

    ```javascript
    // Bad
    if ( condition ) doSomething();

    // Good
    if (condition) {
        doSomething();
    }

    // Bad
    for(var i=0;i<100;i++) someIterativeFn();

    // Good
    var i;

    for (i = 0; i < 100; i++) {
        someIterativeFn();
    }
    ```

    C. Assignments, Declarations
    - Do not expose local variables to the global scope.
    - Use only one `var` per scope (function).
    - `var` statements should always be in the beginning of their respective scope.
    - Prefer literal notation over object notation.

    ```javascript

    // Bad
    var foo = "bar",
        num,
        arr = [1,2,3];

    // Bad
    var foo = "";
    var num = 3;
    var arr = [1,2,3];

    // Good
    var foo = "bar",
        num = 1,
        arr = [1,2,3];

    // also good
    var foo, num, arr;

    // Bad
    var array = new Array(),
        object = new Object();

    // Good
    var array = [],
        object = {};

    // Bad
    function foo() {
      // some statements here

      var bar = "";
    }

    // Good
    function foo() {
      var bar = "";

      // all statements after the variables declarations.
    }

    // Bad
    function foo() {
      bar = "";
      // some statements here
    }

    // Good
    function foo() {
      var bar = "";
      // some statements here
    }

    ```

    D. Functions ( Named, Expression, Constructor )

    ```javascript

    // Named Function Declaration
    function foo(arg1, argN) {

    }

    // Usage
    foo(arg1, argN);

    // Really contrived continuation passing style
    function square(number, callback) {
      callback(number * number);
    }

    square(10, function(square) {
      // callback statements
    });

    // Function Expression
    var square = function(number) {
      // Return something valuable and relevant
      return number * number;
    };

    // UNAPPROVED
    // Useful for functions that call themself, but I think it can be
    // removed from here
    /*
    // Function Expression with Identifier
    // This preferred form has the added value of being
    // able to call itself and have an identity in stack traces:
    var factorial = function factorial(number) {
      if (number < 2) {
        return 1;
      }

      return number * factorial(number - 1);
    };
    */

    // Constructor Declaration
    function FooBar(options) {
      this.options = options;
    }

    // Usage
    var fooBar = new FooBar({a: "alpha"});
    fooBar.options;
    // { a: "alpha" }

    // Functions with callbacks
    foo(function() {
      // Note there is no extra space between the first paren
      // of the executing function call and the word "function"
    });

    ```

3. <a name="type">Type Checking (Courtesy jQuery Core Style Guidelines)</a>

    A. Actual Types

    String:

        typeof variable === "string"

    Number:

        typeof variable === "number"

    Boolean:

        typeof variable === "boolean"

    Object:

        typeof variable === "object"

    Array:

        $.isArray(arrayLikeObject)

    Node:

        elem.nodeType === 1

    null:

        variable === null

    null or undefined (do no use the short version):

        variable == null                                // Bad
        variable === null || variable === undefined     // Good

    undefined:

      Global Variables:

        typeof variable === "undefined"

      Local Variables:

        variable === undefined

      Properties:

        object.prop === undefined
        object.hasOwnProperty(prop)
        "prop" in object

    B. Coerced Types

    Consider the implications of the following...

    Given this HTML:

    ```html

    <input type="text" id="foo-input" value="1">

    ```


    ```javascript

    // 3.B.1.1

    // `foo` has been declared with the value `0` and its type is `number`
    var foo = 0;

    // typeof foo;
    // "number"
    ...

    // Somewhere later in your code, you need to update `foo`
    // with a new value derived from an input element

    foo = document.getElementById("foo-input").value;

    // If you were to test `typeof foo` now, the result would be `string`
    // This means that if you had logic that tested `foo` like:

    if (foo === 1) {
      importantTask();
    }

    // `importantTask()` would never be evaluated, even though `foo` has a value of "1"


    // 3.B.1.2

    // You can preempt issues by using smart coercion with unary + or - operators:

    foo = +document.getElementById("foo-input").value;
    //    ^ unary + operator will convert its right side operand to a number

    // typeof foo;
    // "number"

    if (foo === 1) {
      importantTask();
    }

    // `importantTask()` will be called
    ```

    Here are some common cases along with coercions:


    ```javascript

    // 3.B.2.1

    var number = 1,
      string = "1",
      bool = false;

    number;
    // 1

    number + "";
    // "1"

    string;
    // "1"

    +string;
    // 1

    +string++;
    // 1

    string;
    // 2

    bool;
    // false

    +bool;
    // 0

    bool + "";
    // "false"
    ```


    ```javascript
    // 3.B.2.2

    var number = 1,
      string = "1",
      bool = true;

    string === number;
    // false

    string === number + "";
    // true

    +string === number;
    // true

    bool === number;
    // false

    +bool === number;
    // true

    bool === string;
    // false

    bool === !!string;
    // true
    ```

    ```javascript
    // 3.B.2.3

    var array = ["a", "b", "c"];

    !!~array.indexOf("a");  // true

    // Note that the above should be considered "unnecessarily clever"
    // Prefer the obvious approach of comparing the returned value of
    // indexOf, like:

    if (array.indexOf("a") >= 0) {
      // ...
    }
    ```


4. <a name="cond">Conditional Evaluation</a>

    ```javascript

    // 4.1.1
    // When only evaluating that an array has length,
    // instead of this:
    if (array.length > 0) ...

    // ...evaluate truthiness, like this:
    if (array.length) ...


    // 4.1.2
    // When only evaluating that an array is empty,
    // instead of this:
    if (array.length === 0) ...

    // ...evaluate truthiness, like this:
    if (!array.length) ...


    // 4.1.3
    // When only evaluating that a string is not empty,
    // instead of this:
    if (string !== "") ...

    // ...evaluate truthiness, like this:
    if (string) ...


    // 4.1.4
    // When only evaluating that a string _is_ empty,
    // instead of this:
    if (string === "") ...

    // ...evaluate falsy-ness, like this:
    if (!string) ...


    // 4.1.5
    // When only evaluating that a reference is true,
    // instead of this:
    if (foo === true) ...

    // ...evaluate like you mean it, take advantage of built in capabilities:
    if (foo) ...


    // 4.1.6
    // When evaluating that a reference is false,
    // instead of this:
    if (foo === false) ...

    // ...use negation to coerce a true evaluation
    if (!foo) ...

    // ...Be careful, this will also match: 0, "", null, undefined, NaN
    // If you _MUST_ test for a boolean false, then use
    if (foo === false) ...

    ```
    ALWAYS evaluate for the best, most accurate result - the above is a guideline, not a dogma.

    ```javascript

    // 4.2.1
    // Type coercion and evaluation notes

    // Prefer `===` over `==` (unless the case requires loose type evaluation)

    // === does not coerce type, which means that:

    "1" === 1;
    // false

    // == does coerce type, which means that:

    "1" == 1;
    // true


    // 4.2.2
    // Booleans, Truthies & Falsies

    // Falsy:
    "", 0, null, undefined, NaN, false, void 0

    // Any other return true

    ```

6. <a name="naming">Naming</a>



    A. You are not a human code compiler/compressor, so don't try to be one.

    The following code is an example of egregious naming:

    ```javascript

    // 6.A.1.1
    // Example of code with poor names

    function q(s) {
      return document.querySelectorAll(s);
    }
    var i,a=[],els=q("#foo");
    for(i=0;i<els.length;i++){a.push(els[i]);}
    ```

    Without a doubt, you've written code like this - hopefully that ends today.

    Here's the same piece of logic, but with kinder, more thoughtful naming (and a readable structure):

    ```javascript

    // 6.A.2.1
    // Example of code with improved names

    function query(selector) {
      return document.querySelectorAll(selector);
    }

    var idx = 0,
      elements = [],
      matches = query("#foo"),
      length = matches.length;

    for (idx = 0; idx < length; idx++) {
      elements.push(matches[idx]);
    }

    ```

    A few additional naming pointers:

    ```javascript

    // 6.A.3.1
    // Naming strings

    `dog` is a string


    // 6.A.3.2
    // Naming arrays

    `dogs` is an array of `dog` strings


    // 6.A.3.3
    // Naming functions, objects, instances, etc

    camelCase; function and var declarations


    // 6.A.3.4
    // Naming constructors, prototypes, etc.

    PascalCase; constructor function


    // 6.A.3.5
    // Naming regular expressions

    rDesc = //;


    // 6.A.3.6
    // From the Google Closure Library Style Guide

    functionNamesLikeThis;
    variableNamesLikeThis;
    ConstructorNamesLikeThis;
    EnumNamesLikeThis;
    methodNamesLikeThis;
    SYMBOLIC_CONSTANTS_LIKE_THIS;

    ```

    B. Faces of `this`


    ```

    As a last resort, create an alias to `this` using `self` as an Identifier. This is extremely bug prone and should be avoided whenever possible.

    ```javascript

    // 6.B.3

    function Device( opts ) {
      var self = this;

      this.value = null;

      stream.read( opts.path, function( data ) {

        self.value = data;

      });

      setInterval(function() {

        self.emit("event");

      }, opts.freq || 100 );
    }

    ```

7. <a name="misc">Misc</a>

    This section will serve to illustrate ideas and concepts that should not be considered dogma, but instead exists to encourage questioning practices in an attempt to find better ways to do common JavaScript programming tasks.

    A. Using `switch` should be avoided, modern method tracing will blacklist functions with switch statements

    There seems to be drastic improvements to the execution of `switch` statements in latest releases of Firefox and Chrome.
    http://jsperf.com/switch-vs-object-literal-vs-module

    Notable improvements can be witnesses here as well:
    https://github.com/rwldrn/idiomatic.js/issues/13

    ```javascript

    // 7.A.1.1
    // An example switch statement

    switch( foo ) {
      case "alpha":
        alpha();
        break;
      case "beta":
        beta();
        break;
      default:
        // something to default to
        break;
    }

    // 7.A.1.2
    // A alternate approach that supports composability and reusability is to
    // use an object to store "cases" and a function to delegate:

    var cases, delegator;

    // Example returns for illustration only.
    cases = {
      alpha: function() {
        // statements
        // a return
        return [ "Alpha", arguments.length ];
      },
      beta: function() {
        // statements
        // a return
        return [ "Beta", arguments.length ];
      },
      _default: function() {
        // statements
        // a return
        return [ "Default", arguments.length ];
      }
    };

    delegator = function() {
      var args, key, delegate;

      // Transform arguments list into an array
      args = [].slice.call( arguments );

      // shift the case key from the arguments
      key = args.shift();

      // Assign the default case handler
      delegate = cases._default;

      // Derive the method to delegate operation to
      if ( cases.hasOwnProperty( key ) ) {
        delegate = cases[ key ];
      }

      // The scope arg could be set to something specific,
      // in this case, |null| will suffice
      return delegate.apply( null, args );
    };

    // 7.A.1.3
    // Put the API in 7.A.1.2 to work:

    delegator( "alpha", 1, 2, 3, 4, 5 );
    // [ "Alpha", 5 ]

    // Of course, the `case` key argument could easily be based
    // on some other arbitrary condition.

    var caseKey, someUserInput;

    // Possibly some kind of form input?
    someUserInput = 9;

    if ( someUserInput > 10 ) {
      caseKey = "alpha";
    } else {
      caseKey = "beta";
    }

    // or...

    caseKey = someUserInput > 10 ? "alpha" : "beta";

    // And then...

    delegator( caseKey, someUserInput );
    // [ "Beta", 1 ]

    // And of course...

    delegator();
    // [ "Default", 0 ]


    ```

    B. Early returns promote code readability with negligible performance difference

    ```javascript

    // 7.B.1.1
    // Bad:
    function returnLate(foo) {
      var ret;

      if (foo) {
        ret = "foo";
      } else {
        ret = "quux";
      }
      return ret;
    }

    // Good:

    function returnEarly(foo) {

      if (foo) {
        return "foo";
      }
      return "quux";
    }

    ```


8. <a name="native">Native & Host Objects</a>

    The basic principle here is:

    ### Don't do stupid shit and everything will be ok.

    To reinforce this concept, please watch the following presentation:

    #### “Everything is Permitted: Extending Built-ins” by Andrew Dupont (JSConf2011, Portland, Oregon)

    <iframe src="http://blip.tv/play/g_Mngr6LegI.html" width="480" height="346" frameborder="0" allowfullscreen></iframe><embed type="application/x-shockwave-flash" src="http://a.blip.tv/api.swf#g_Mngr6LegI" style="display:none"></embed>

    http://blip.tv/jsconf/jsconf2011-andrew-dupont-everything-is-permitted-extending-built-ins-5211542


9. <a name="comments">Comments</a>

    #### Single line above the code that is subject
    #### Multiline is good
    #### End of line comments are prohibited!
    #### JSDoc style is good, but requires a significant time investment


10. <a name="language">One Language Code</a>

    Programs should be written in one language, whatever that language may be, as dictated by the maintainer or maintainers.

## Appendix

### JSHint file

// code

----------


<a rel="license" href="http://creativecommons.org/licenses/by/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by/3.0/80x15.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Principles of Writing Consistent, Idiomatic JavaScript</span> by <a xmlns:cc="http://creativecommons.org/ns#" href="https://github.com/rwldrn/idiomatic.js" property="cc:attributionName" rel="cc:attributionURL">Rick Waldron and Contributors</a> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/deed.en_US">Creative Commons Attribution 3.0 Unported License</a>.<br />Based on a work at <a xmlns:dct="http://purl.org/dc/terms/" href="https://github.com/rwldrn/idiomatic.js" rel="dct:source">github.com/rwldrn/idiomatic.js</a>.
