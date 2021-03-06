# JavaScript Style Guide

*A mostly reasonable approach to JavaScript*

### Table of Contents

<!-- MarkdownTOC -->

- [Best Practices](#best-practices)
    - [Semicolons`;`](#semicolons)
    - [Style Checking](#style-checking)
    - [Linting](#linting)
    - [Events](#events)
    - [Modules](#modules)
    - [jQuery](#jquery)
    - [ECMAScript 5 Compatibility](#es5)
    - [Testing](#testing)
    - [`console` statements](#console statements)
    - [Comments](#comments)
    - [Variable Naming](#variable-naming)
    - [Polyfills](#polyfills)
    - [Everyday Tricks](#everyday-tricks)
    - [Performance](#performance)
- [Syntax](#syntax)
    - [Types](#types)
    - [Objects](#objects)
    - [Arrays](#arrays)
    - [Strings](#strings)
    - [Functions](#functions)
    - [Properties](#properties)
    - [Variables](#variables)
    - [Hoisting](#hoisting)
    - [Conditional Expressions & Equality](#conditional-expressions)
    - [Blocks](#blocks)
    - [Comments](#comments-1)
    - [Whitespace](#whitespace)
    - [Commas](#commas)
    - [Semicolons](#semicolons-1)
    - [Type Casting & Coercion](#type-casting--coercion)
    - [Naming Conventions](#naming-conventions)
    - [Accessors](#accessors)
    - [Constructors](#constructors)
- [Resources](#resources)
    - [Read This](#read-this)
    - [Other Styleguides](#other-styleguides)
    - [Other Styles](#other-styles)
    - [Further Reading](#further-readingj)
    - [Books](#books)
    - [Blogs](#blogs)
- [License](#license)

<!-- /MarkdownTOC -->


## Best Practices
### Semicolons`;`

Automatic Semicolon Insertion _(ASI)_ is a [complicated feature](http://www.2ality.com/2011/05/semicolon-insertion.html). But, writing a `;` at the end of every line is painful. And the rules aren't that complex to learn. Don't bother with the semi-colons.

### Style Checking

Tools like [jscs](https://github.com/jscs-dev/node-jscs) can strictly enforce a code style. This makes it easy to ensure that developers can look at code they didn't write and more quickly understand what's going on. However, style-checking failures can be painful to deal with. Leave the implementation up to the team working the the project.

Generally, these work well in small projects. A sample `.jscrc` is in this repo.

### Linting

Linting is **always a great idea**. Don't use a linter that's super opinionated about how the code should be styled, like [`jslint`](http://www.jslint.com/). Instead, use something more lenient like [`jshint`](http://www.jshint.com/) or [`eslint`](https://github.com/eslint/eslint).

#### A few tips when using JSHint

- Declare a `.jshintignore` file and include `node_modules`, `bower_components`, and the like
- You should use a `.jshintrc` file to keep your rules together. See the one in this repo.

### Events

  - When attaching data payloads to events (whether DOM events or something more proprietary like Backbone events), pass a hash instead of a raw value. This allows a subsequent contributor to add more data to the event payload without finding and updating every handler for the event. For example, instead of:

    ```js
    // bad
    $(this).trigger('listingUpdated', listing.id)

    …

    $(this).on('listingUpdated', function(e, listingId){
      // do something with listingId
    })
    ```

    prefer:

    ```js
    // good
    $(this).trigger('listingUpdated', { listingId : listing.id })

    …

    $(this).on('listingUpdated', function(e, data){
      // do something with data.listingId
    })
    ```

### Modules

  - write all modules in the CommonJs style. We'll use browserify to turn it into a UMD module where necessary.
  - The file should be named with camelCase, live in a folder with the same name, and match the name of the single export.
  - Add a method called noConflict() that sets the exported module to the previous version and returns this one.
  - Always declare `'use strict';` at the top of the module.

    ```javascript
    // fancyInput/fancyInput.js
    'use strict'

    var previousFancyInput = global.FancyInput

    function FancyInput(options){
      this.options = options || {}
    }

    FancyInput.noConflict = function noConflict(){
      global.FancyInput = previousFancyInput
      return FancyInput
    }

    module.exports = FancyInput
    ```

### jQuery

  - Prefix jQuery object variables with a `$`.

    ```javascript
    // bad
    var sidebar = $('.sidebar')

    // good
    var $sidebar = $('.sidebar')
    ```

  - Cache jQuery lookups.

    ```javascript
    // bad
    function setSidebar(){
      $('.sidebar').hide()

      // …stuff…

      $('.sidebar').css({
        'background-color': 'pink'
      })
    }

    // good
    function setSidebar(){
      var $sidebar = $('.sidebar')
      $sidebar.hide()

      // …stuff…

      $sidebar.css({
        'background-color': 'pink'
      })
    }
    ```

  - For DOM queries use Cascading `$('.sidebar ul')` or parent > child `$('.sidebar > ul')`. [jsPerf](http://jsperf.com/jquery-find-vs-context-sel/16)
  - Use `find` with scoped jQuery object queries.

    ```javascript
    // bad
    $('ul', '.sidebar').hide()

    // bad
    $('.sidebar').find('ul').hide()

    // good
    $('.sidebar ul').hide()

    // good
    $('.sidebar > ul').hide()

    // good
    $sidebar.find('ul')
    ```

### ECMAScript 5 Compatibility [es5]

  - Refer to [Kangax](https://twitter.com/kangax/)'s ES5 [compatibility table](http://kangax.github.com/es5-compat-table/)
  - Note that you're probably using a library that includes [es5shim](https://github.com/kriskowal/es5-shim), this means that you can probably freely use es5 methods.

### Testing

  - **Yup.**

    ```javascript
    function(){
      return true
    }
    ```

### `console` statements [console statements]

Preferably bake `console` statements into a service that can easily be disabled in production. Alternatively, don't ship any `console.log` printing statements to production distributions.

### Comments

Comments **aren't meant to explain what** the code does. Good **code is supposed to be self-explanatory**. If you're thinking of writing a comment to explain what a piece of code does, chances are you need to change the code itself. The exception to that rule is explaining what a regular expression does. Good comments are supposed to **explain why** code does something that may not seem to have a clear-cut purpose.

#### Bad

```js
// create the centered container
var p = $('<p/>')
p.center(div)
p.text('foo')
```

#### Good

```js
var $container = $('<p/>')
  , contents = 'foo'

$container.center(parent)
$container.text(contents)

megaphone.on('data', function(value){
  // the megaphone periodically emits updates for container
  container.text(value)
})
```

```js
var numeric = /\d+/ // one or more digits somewhere in the string
if (numeric.test(text)){
  console.log('so many numbers!')
}
```

Commenting out entire blocks of code _should be avoided entirely_, that's why you have version control systems in place!

### Variable Naming

Variables must have meaningful names so that you don't have to resort to commenting what a piece of functionality does. Instead, try to be expressive while succinct, and use meaningful variable names.

#### Bad

```js
function a (x, y, z) {
  return z * y / x
}
a(4, 2, 6);
// <- 3
```

#### Good

```js
function ruleOfThree (had, got, have) {
  return have * got / had
}
ruleOfThree(4, 2, 6)
// <- 3
```

### Polyfills

Where possible use the native browser implementation and include [a polyfill that provides that behavior](http://remysharp.com/2010/10/08/what-is-a-polyfill/) for unsupported browsers. This makes the code easier to work with and less involved in hackery to make things just work.

If you can't patch a piece of functionality with a polyfill, then [wrap all uses of the patching code](http://blog.ponyfoo.com/2014/08/05/building-high-quality-front-end-modules) in a globally available method that is accessible from everywhere in the application.

### Everyday Tricks

Use `||` to define a default value. If the left-hand value is [falsy][29] then the right-hand value will be used.

```js
function a(value){
  var defaultValue = 33
  var used = value || defaultValue
}
```

Use `.bind` to [partially-apply](http://ejohn.org/blog/partial-functions-in-javascript/) functions.

```js
var sum = function sum(a, b){
    return a + b
  }
  , addSeven = sum.bind(null, 7)

addSeven(6)
// <- 13
```

Use `Array.prototype.slice.call` to cast array-like objects to true arrays.

```js
var args = [].slice.call(arguments)
```

Use [event emitters](https://github.com/bevacqua/contra#%CE%BBemitterthing-options) on all the things!

```js
var emitter = new Backbone.Events()

$body.on('click', function onBodyClick(e){
  emitter.trigger('click', e.target)
})

emitter.on('click', function onEmitterClick(elem) {
  console.log(elem)
})

// simulate click
emitter.trigger('click', document.body)
```

Use `Function()` as a _"no-op"_.

```js
function onThingHappended(cb){
  setTimeout(cb || Function(), 2000)
}
```

### Performance

  - [On Layout & Web Performance](http://kellegous.com/j/2013/01/26/layout-performance/)
  - [String vs Array Concat](http://jsperf.com/string-vs-array-concat/2)
  - [Try/Catch Cost In a Loop](http://jsperf.com/try-catch-in-loop-cost)
  - [Bang Function](http://jsperf.com/bang-function)
  - [jQuery Find vs Context, Selector](http://jsperf.com/jquery-find-vs-context-sel/13)
  - [innerHTML vs textContent for script text](http://jsperf.com/innerhtml-vs-textcontent-for-script-text)
  - [Long String Concatenation](http://jsperf.com/ya-string-concat)
  - Loading…



## Syntax
### Types

  - **Primitives**: When you access a primitive type you work directly on its value

    + `string`
    + `number`
    + `boolean`
    + `null`
    + `undefined`

    ```javascript
    var foo = 1
      , bar = foo

    bar = 9

    console.log(foo, bar) // => 1, 9
    ```
  - **Complex**: When you access a complex type you work on a reference to its value

    + `object`
    + `array`
    + `function`

    ```javascript
    var foo = [1, 2]
      , bar = foo

    bar[0] = 9

    console.log(foo[0], bar[0]) // => 9, 9
    ```


### Objects

  - Use the literal syntax for object creation.

    ```javascript
    // bad
    var item = new Object()

    // good
    var item = {}
    ```

  - Don't use [reserved words](http://es5.github.io/#x7.6.1) as keys. It won't work in IE8. [More info](https://github.com/airbnb/javascript/issues/61)

    ```javascript
    // bad
    var superman = {
      default: {clark: 'kent'}
      , private: true
    }

    // good
    var superman = {
      defaults: {clark: 'kent'}
      , hidden: true
    }
    ```

  - Use readable synonyms in place of reserved words.

    ```javascript
    // bad
    var superman = {
      class: 'alien'
    }

    // bad
    var superman = {
      klass: 'alien'
    }

    // good
    var superman = {
      type: 'alien'
    }
    ```

### Arrays

  - Use the literal syntax for array creation

    ```javascript
    // bad
    var items = new Array()

    // good
    var items = []
    ```

  - If you don't know array length use Array#push.

    ```javascript
    var someStack = []

    // bad
    someStack[someStack.length] = 'abracadabra'

    // good
    someStack.push('abracadabra')
    ```

  - When you need to copy an array use Array#slice. [jsPerf](http://jsperf.com/converting-arguments-to-an-array/7)

    ```javascript
    var len = items.length
      , itemsCopy = []
      , i

    // bad
    for (i = 0; i < len; i++){
      itemsCopy[i] = items[i]
    }

    // good
    itemsCopy = items.slice()
    ```

  - To convert an array-like object to an array, use Array#slice.

    ```javascript
    function trigger(){
      var args = Array.prototype.slice.call(arguments)
      …
    }
    ```



### Strings

  - Use single quotes `''` for strings

    ```javascript
    // bad
    var name = "Bob Parr"

    // good
    var name = 'Bob Parr'

    // bad
    var fullName = "Bob " + this.lastName

    // good
    var fullName = 'Bob ' + this.lastName
    ```

  - Strings longer than 80 characters should be written across multiple lines using string concatenation.
  - Note: If overused, long strings with concatenation could impact performance. [jsPerf](http://jsperf.com/ya-string-concat) & [Discussion](https://github.com/airbnb/javascript/issues/40)

    ```javascript
    // bad
    var errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.'

    // bad
    var errorMessage = 'This is a super long error that \
    was thrown because of Batman. \
    When you stop to think about \
    how Batman had anything to do \
    with this, you would get nowhere \
    fast.'

    // good
    var errorMessage = 'This is a super long error that ' +
      'was thrown because of Batman. ' +
      'When you stop to think about ' +
      'how Batman had anything to do ' +
      'with this, you would get nowhere ' +
      'fast.'
    ```

  - When programatically building up a string, use Array#join instead of string concatenation. Mostly for IE: [jsPerf](http://jsperf.com/string-vs-array-concat/2).

    ```javascript
    var items
      , messages
      , length
      , i

    messages = [{
      state: 'success'
      , message: 'This one worked.'
    },{
      state: 'success'
      , message: 'This one worked as well.'
    },{
        state: 'error',
        , message: 'This one did not work.'
    }]

    length = messages.length

    // bad
    function inbox(messages){
      items = '<ul>'

      for (i = 0; i < length; i++){
        items += '<li>' + messages[i].message + '</li>'
      }

      return items + '</ul>'
    }

    // good
    function inbox(messages){
      items = []

      for (i = 0; i < length; i++){
        items[i] = messages[i].message
      }

      return '<ul><li>' + items.join('</li><li>') + '</li></ul>'
    }
    ```



### Functions

  - Function expressions:

    ```javascript
    // anonymous function expression
    var anonymous = function(){
      return true
    }

    // named function expression
    var named = function named(){
      return true
    }

    // immediately-invoked function expression (IIFE)
    (function(){
      console.log('Welcome to the Internet. Please follow me.')
    })()
    ```

  - Never declare a function in a non-function block (if, while, etc). Assign the function to a variable instead. Browsers will allow you to do it, but they all interpret it differently, which is bad news bears.
  - **Note:** ECMA-262 defines a `block` as a list of statements. A function declaration is not a statement. [Read ECMA-262's note on this issue](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf#page=97).

    ```javascript
    // bad
    if (currentUser){
      function test(){
        console.log('Nope.')
      }
    }

    // good
    var test
    if (currentUser){
      test = function test(){
        console.log('Yup.')
      }
    }
    ```

  - Never name a parameter `arguments`, this will take precedence over the `arguments` object that is given to every function scope.

    ```javascript
    // bad
    function nope(name, options, arguments){
      // …stuff…
    }

    // good
    var yup = function yup(name, options, args){
      // …stuff…
    }
    ```

  - **Always prefer named functions** This makes debugging much easier.
  - You should assign a function to a variable. This makes hoisting easier to reason about.

  - Never put spaces immediately after an opening parens or immediately before a closing parens.

      ```javascript
      // bad
      var bad = function ( badness, devils, misery ) {

      }

      // good
      var good = function good(happiness, heaven, cheesecake){

      }
      ```

  - Avoid creating functions that accept more than 3 arguments. You should probably pass a single options argument instead.

      ```javascript
      // bad
      var manyThings = function manyThings(first, second, moar, evenMoar){

      }

      // good
      var manyThings = function manyThings(options){
        /*
        options.first
        options.second
        etc…
        */
      }
      ```




### Properties

  - Use dot notation when accessing properties.

    ```javascript
    var luke = {
      jedi: true
      , age: 28
    }

    // bad
    var isJedi = luke['jedi']

    // good
    var isJedi = luke.jedi
    ```

  - Use subscript notation `[]` when accessing properties with a variable.

    ```javascript
    var luke = {
      jedi: true
      , age: 28
    }

    function getProp(prop){
      return luke[prop]
    }

    var isJedi = getProp('jedi')
    ```



### Variables

  - Always use `var` to declare variables. Not doing so will result in global variables. We want to avoid polluting the global namespace. Captain Planet warned us of that.

    ```javascript
    // bad
    superPower = new SuperPower()

    // good
    var superPower = new SuperPower()
    ```

  - Use one `var` declaration for multiple variables and declare each variable on a newline.

    ```javascript
    // bad
    var items = getItems()
    var goSportsTeam = true
    var dragonball = 'z'

    // good
    var items = getItems()
      , goSportsTeam = true
      , dragonball = 'z'
    ```

  - Declare unassigned variables last. This is helpful when later on you might need to assign a variable depending on one of the previous assigned variables. You should give "unassigned" variables a default value. This allows modern JS engines like V8 to better optimize. It also makes the code more readable.

    ```javascript
    // bad
    var i, len, dragonball
      , items = getItems()
      , goSportsTeam = true

    // bad
    var i, items = getItems()
      , dragonball
      , goSportsTeam = true
      , len

    // good
    var items = getItems()
      , goSportsTeam = true
      , dragonball = {}
      , length = 0
      , i = 0
    ```

  - Assign variables at the top of their scope. This helps avoid issues with variable declaration and assignment hoisting related issues.

    ```javascript
    // bad
    function(){
      test()
      console.log('doing stuff..')

      //..other stuff..

      var name = getName()

      if (name === 'test'){
        return false
      }

      return name
    }

    // good
    function(){
      var name = getName()

      test()
      console.log('doing stuff..')

      //..other stuff..

      if (name === 'test'){
        return false
      }

      return name
    }

    // bad
    function(){
      var name = getName()

      if (!arguments.length){
        return false
      }

      return true
    }

    // good
    function(){
      if (!arguments.length){
        return false
      }

      var name = getName()

      return true
    }
    ```



### Hoisting

  - Variable declarations get hoisted to the top of their scope, their assignment does not.

    ```javascript
    // we know this wouldn't work (assuming there
    // is no notDefined global variable)
    function example(){
      console.log(notDefined) // => throws a ReferenceError
    }

    // creating a variable declaration after you
    // reference the variable will work due to
    // variable hoisting. Note: the assignment
    // value of `true` is not hoisted.
    function example(){
      console.log(declaredButNotAssigned) // => undefined
      var declaredButNotAssigned = true
    }

    // The interpreter is hoisting the variable
    // declaration to the top of the scope.
    // Which means our example could be rewritten as:
    function example(){
      var declaredButNotAssigned
      console.log(declaredButNotAssigned) // => undefined
      declaredButNotAssigned = true
    }
    ```

  - Anonymous function expressions hoist their variable name, but not the function assignment.

    ```javascript
    function example(){
      console.log(anonymous) // => undefined

      anonymous() // => TypeError anonymous is not a function

      var anonymous = function(){
        console.log('anonymous function expression')
      }
    }
    ```

  - Named function expressions hoist the variable name, not the function name or the function body.

    ```javascript
    function example(){
      console.log(named) // => undefined

      named() // => TypeError named is not a function

      superPower() // => ReferenceError superPower is not defined

      var named = function superPower(){
        console.log('Flying')
      }
    }

    // the same is true when the function name
    // is the same as the variable name.
    function example(){
      console.log(named) // => undefined

      named() // => TypeError named is not a function

      var named = function named(){
        console.log('named')
      }
    }
    ```

  - Function declarations hoist their name and the function body.

    ```javascript
    function example(){
      superPower() // => Flying

      function superPower(){
        console.log('Flying')
      }
    }
    ```

  - For more information refer to [JavaScript Scoping & Hoisting](http://www.adequatelygood.com/2010/2/JavaScript-Scoping-and-Hoisting) by [Ben Cherry](http://www.adequatelygood.com/)




### Conditional Expressions & Equality [conditional-expressions]

  - Use `===` and `!==` over `==` and `!=`.
  - Conditional expressions are evaluated using coercion with the `ToBoolean` method and always follow these simple rules:

    + **Objects** evaluate to **true**
    + **Undefined** evaluates to **false**
    + **Null** evaluates to **false**
    + **Booleans** evaluate to **the value of the boolean**
    + **Numbers** evaluate to **false** if **+0, -0, or NaN**, otherwise **true**
    + **Strings** evaluate to **false** if an empty string `''`, otherwise **true**

    ```javascript
    if ([0]){
      // true
      // An array is an object, objects evaluate to true
    }
    ```

  - Use shortcuts.

    ```javascript
    // bad
    if (name !== ''){
      // …stuff…
    }

    // good
    if (name){
      // …stuff…
    }

    // bad
    if (collection.length > 0){
      // …stuff…
    }

    // good
    if (collection.length){
      // …stuff…
    }
    ```

  - For more information see [Truth Equality and JavaScript](http://javascriptweblog.wordpress.com/2011/02/07/truth-equality-and-javascript/#more-2108) by Angus Croll



### Blocks

  - Single line blocks don't require braces, but should be easy to read. If your single line extends more than 80 characters, break into two lines, and indent the contents of the block.

    ```javascript
    // bad
    if (test){
      return false
    }

    // bad
    if (test)
      return false

    // good
    if (test) return false


    // bad
    if (test && test.arg === doSomethingWith(5) && puppies() === 'cute') ciderHouse = true

    // good
    if (test && test.arg === doSomethingWith(5) && puppies() === 'cute')
      ciderHouse = true

    // bad
    function(){ return false }

    // good
    function(){
      return false
    }
    ```



### Comments

  - Use `/** … */` for multiline comments. Include a description, specify types and values for all parameters and return values.

    ```javascript
    // bad
    // make() returns a new element
    // based on the passed in tag name
    //
    // @param <String> tag
    // @return <Element> element
    function make(tag){

      // …stuff…

      return element
    }

    // good
    /**
     * make() returns a new element
     * based on the passed in tag name
     *
     * @param <String> tag
     * @return <Element> element
     */
    function make(tag){

      // …stuff…

      return element
    }
    ```

  - Use `//` for single line comments. Place single line comments on a newline above the subject of the comment. Put an empty line before the comment.

    ```javascript
    // bad
    var active = true;  // is current tab

    // good
    // is current tab
    var active = true

    // bad
    function getType(){
      console.log('fetching type…')
      // set the default type to 'no type'
      var type = this._type || 'no type'

      return type
    }

    // good
    function getType(){
      console.log('fetching type…')

      // set the default type to 'no type'
      var type = this._type || 'no type'

      return type
    }
    ```

  - Prefixing your comments with `FIXME` or `TODO` helps other developers quickly understand if you're pointing out a problem that needs to be revisited, or if you're suggesting a solution to the problem that needs to be implemented. These are different than regular comments because they are actionable. The actions are `FIXME -- need to figure this out` or `TODO -- need to implement`.

  - Use `// FIXME:` to annotate problems

    ```javascript
    function Calculator(){

      // FIXME: shouldn't use a global here
      total = 0

      return this
    }
    ```

  - Use `// TODO:` to annotate solutions to problems

    ```javascript
    function Calculator(){

      // TODO: total should be configurable by an options param
      this.total = 0

      return this
    }
  ```



### Whitespace

  - Use soft tabs set to 2 spaces

    ```javascript
    // bad
    function(){
    ∙∙∙∙var name
    }

    // bad
    function(){
    ∙var name
    }

    // good
    function(){
    ∙∙var name
    }
    ```

  - Place no spaces before the leading brace.

    ```javascript
    // bad
    function test() {
      console.log('test')
    }

    // good
    function test(){
      console.log('test')
    }

    // bad
    dog.set('attr',{
      age: '1 year',
      breed: 'Bernese Mountain Dog'
    })

    // good
    dog.set('attr', {
      age: '1 year',
      breed: 'Bernese Mountain Dog'
    })
    ```

  - Set off operators with spaces.

    ```javascript
    // bad
    var x=y+5

    // good
    var x = y + 5
    ```

  - Place an empty newline at the end of the file.

    ```javascript
    // bad
    (function(global){
      // …stuff…
    })(this)
    ```

    ```javascript
    // good
    (function(global){
      // …stuff…
    })(this)

    ```

  - Use indentation when making long method chains.

    ```javascript
    // bad
    $('#items').find('.selected').highlight().end().find('.open').updateCount()

    // good
    $('#items')
      .find('.selected')
        .highlight()
        .end()
      .find('.open')
        .updateCount()

    // bad
    var leds = stage.selectAll('.led').data(data).enter().append('svg:svg').class('led', true)
        .attr('width',  (radius + margin) * 2).append('svg:g')
        .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
        .call(tron.led)

    // good
    var leds = stage.selectAll('.led')
        .data(data)
      .enter().append('svg:svg')
        .class('led', true)
        .attr('width',  (radius + margin) * 2)
      .append('svg:g')
        .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
        .call(tron.led)
    ```


### Commas

  - Leading commas: **Hell yes.**
  - [Read why comma first is better](https://gist.github.com/isaacs/357981/)

    ```javascript
    // bad
    var once,
        upon,
        aTime

    // good
    var once
      , upon
      , aTime

    // bad
    var hero = {
      firstName: 'Bob',
      lastName: 'Parr',
      heroName: 'Mr. Incredible',
      superPower: 'strength'
    }

    // good
    var hero = {
        firstName: 'Bob'
      , lastName: 'Parr'
      , heroName: 'Mr. Incredible'
      , superPower: 'strength'
    }
    ```

  - Additional trailing comma: **Nope.** This can cause problems with IE6/7 and IE9 if it's in quirksmode. Also, in some implementations of ES3 would add length to an array if it had an additional trailing comma. This was clarified in ES5 ([source](http://es5.github.io/#D)):

  > Edition 5 clarifies the fact that a trailing comma at the end of an ArrayInitialiser does not add to the length of the array. This is not a semantic change from Edition 3 but some implementations may have previously misinterpreted this.

    ```javascript
    // bad
    var hero = {
      firstName: 'Kevin'
      , lastName: 'Flynn'
      ,
    }

    var heroes = [
      'Batman'
      , 'Superman'
      ,
    ]

    // good
    var hero = {
      firstName: 'Kevin'
      , lastName: 'Flynn'
    }

    var heroes = [
      'Batman'
      , 'Superman'
    ]
    ```



### Semicolons

  - **NO.**
  - ASI means that you almost never need semicolons. Leave 'em out and live free! JsHint will catch nearly any error you might make.

    ```javascript
    // bad
    (function(){
      var name = 'Skywalker';
      return name;
    })();

    // good
    (function(){
      var name = 'Skywalker'
      return name
    })()

    // good
    var a

    ;(function(){
      var name = 'Skywalker'
      return name
    })()
    ```

### Type Casting & Coercion

  - Perform type coercion at the beginning of the statement.
  - Strings:

    ```javascript
    //  => this.reviewScore = 9

    // bad
    var totalScore = this.reviewScore + ''

    // good
    var totalScore = '' + this.reviewScore

    // bad
    var totalScore = '' + this.reviewScore + ' total score'

    // good
    var totalScore = this.reviewScore + ' total score'
    ```

  - Use `parseInt` for Numbers and always with a radix for type casting.

    ```javascript
    var inputValue = '4'

    // bad
    var val = new Number(inputValue)

    // bad
    var val = +inputValue

    // bad
    var val = inputValue >> 0

    // bad
    var val = parseInt(inputValue)

    // good
    var val = Number(inputValue)

    // better
    var val = parseInt(inputValue, 10)
    ```

  - If for whatever reason you are doing something wild and `parseInt` is your bottleneck and need to use Bitshift for [performance reasons](http://jsperf.com/coercion-vs-casting/3), leave a comment explaining why and what you're doing.
  - **Note:** Be careful when using bitshift operations. Numbers are represented as [64-bit values](http://es5.github.io/#x4.3.19), but Bitshift operations always return a 32-bit integer ([source](http://es5.github.io/#x11.7)). Bitshift can lead to unexpected behavior for integer values larger than 32 bits. [Discussion](https://github.com/airbnb/javascript/issues/109)

    ```javascript
    // good
    /**
     * parseInt was the reason my code was slow.
     * Bitshifting the String to coerce it to a
     * Number made it a lot faster.
     */
    var val = inputValue >> 0
    ```

  - Booleans:

    ```javascript
    var age = 0

    // bad
    var hasAge = new Boolean(age)

    // good
    var hasAge = Boolean(age)

    // better
    var hasAge = !!age
    ```



### Naming Conventions

  - Avoid single letter names. Be descriptive with your naming.

    ```javascript
    // bad
    function q(){
      // …stuff…
    }

    // good
    function query(){
      // …stuff…
    }
    ```

  - Use camelCase when naming objects, functions, and instances

    ```javascript
    // bad
    var OBJEcttsssss = {}
    var this_is_my_object = {}
    function c(){}
    var u = new user({
      name: 'Bob Parr'
    })

    // good
    var thisIsMyObject = {}
    function thisIsMyFunction(){}
    var user = new User({
      name: 'Bob Parr'
    })
    ```

  - Use PascalCase when naming constructors or classes

    ```javascript
    // bad
    function user(options){
      this.name = options.name
    }

    var bad = new user({
      name: 'nope'
    })

    // good
    function User(options){
      this.name = options.name
    }

    var good = new User({
      name: 'yup'
    })
    ```

  - Use a leading underscore `_` when naming private properties

    ```javascript
    // bad
    this.__firstName__ = 'Panda'
    this.firstName_ = 'Panda'

    // good
    this._firstName = 'Panda'
    ```

  - When saving a reference to `this` use `self`. Avoid using self unless absolutely necessary. Pefer `_.bind()`

    ```javascript
    // bad
    function(){
      var that = this
      return function(){
        console.log(that)
      }
    }

    // bad
    function(){
      var _this = this
      return function(){
        console.log(_this)
      }
    }

    // bad
    function(){
      var self = this
      self.thing = true
    }

    // good
    function(){
      var self = this
      this.thing = true
      return function(){
        console.log(self)
      }
    }
    ```

  - Name your functions. This is helpful for stack traces.

    ```javascript
    // bad
    var log = function(msg){
      console.log(msg)
    }

    // good
    var log = function log(msg){
      console.log(msg)
    }
    ```



### Accessors

  - Accessor functions for properties are not required
  - If you do make accessor functions use getVal() and setVal('hello')

    ```javascript
    // bad
    dragon.age()

    // good
    dragon.getAge()

    // bad
    dragon.age(25)

    // good
    dragon.setAge(25)
    ```

  - If the property is a boolean, use isVal() or hasVal()

    ```javascript
    // bad
    if (!dragon.age()){
      return false
    }

    // good
    if (!dragon.hasAge()){
      return false
    }
    ```

  - It's okay to create get() and set() functions, but be consistent.

    ```javascript
    function Jedi(options){
      options || (options = {})
      var lightsaber = options.lightsaber || 'blue'
      this.set('lightsaber', lightsaber)
    }

    Jedi.prototype.set = function(key, val){
      this[key] = val
    }

    Jedi.prototype.get = function(key){
      return this[key]
    }
    ```



### Constructors

  - Assign methods to the prototype object, instead of overwriting the prototype with a new object. Overwriting the prototype makes inheritance impossible: by resetting the prototype you'll overwrite the base!

    ```javascript
    function Jedi(){
      console.log('new jedi')
    }

    // bad
    Jedi.prototype = {
      fight: function fight(){
        console.log('fighting')
      },

      block: function block(){
        console.log('blocking')
      }
    }

    // good
    Jedi.prototype.fight = function fight(){
      console.log('fighting')
    }

    Jedi.prototype.block = function block(){
      console.log('blocking')
    }
    ```

  - Methods can return `this` to help with method chaining.

    ```javascript
    // bad
    Jedi.prototype.jump = function(){
      this.jumping = true
      return true
    }

    Jedi.prototype.setHeight = function(height){
      this.height = height
    }

    var luke = new Jedi()
    luke.jump() // => true
    luke.setHeight(20) // => undefined

    // good
    Jedi.prototype.jump = function(){
      this.jumping = true
      return this
    }

    Jedi.prototype.setHeight = function(height){
      this.height = height
      return this
    }

    var luke = new Jedi()

    luke.jump()
      .setHeight(20)
    ```


  - It's okay to write a custom toString() method, just make sure it works successfully and causes no side effects.

    ```javascript
    function Jedi(options){
      options || (options = {})
      this.name = options.name || 'no name'
    }

    Jedi.prototype.getName = function getName(){
      return this.name
    }

    Jedi.prototype.toString = function toString(){
      return 'Jedi - ' + this.getName()
    }
    ```



## Resources


### Read This [read-this]

  - [Annotated ECMAScript 5.1](http://es5.github.com/)

### Other Styleguides [other-styleguides]

  - [Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
  - [jQuery Core Style Guidelines](http://docs.jquery.com/JQuery_Core_Style_Guidelines)
  - [Principles of Writing Consistent, Idiomatic JavaScript](https://github.com/rwldrn/idiomatic.js/)
  - This style guide is based on [AirBnB's style guide](https://github.com/airbnb/javascript)

### Other Styles [other-styles]

  - [Naming this in nested functions](https://gist.github.com/4135065) - Christian Johansen
  - [Conditional Callbacks](https://github.com/airbnb/javascript/issues/52)
  - [Popular JavaScript Coding Conventions on Github](http://sideeffect.kr/popularconvention/#javascript)

### Further Reading [further-readingj]

  - [Understanding JavaScript Closures](http://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/) - Angus Croll
  - [Basic JavaScript for the impatient programmer](http://www.2ality.com/2013/06/basic-javascript.html) - Dr. Axel Rauschmayer

### Books

  - [JavaScript: The Good Parts](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742) - Douglas Crockford
  - [JavaScript Patterns](http://www.amazon.com/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752) - Stoyan Stefanov
  - [Pro JavaScript Design Patterns](http://www.amazon.com/JavaScript-Design-Patterns-Recipes-Problem-Solution/dp/159059908X)  - Ross Harmes and Dustin Diaz
  - [High Performance Web Sites: Essential Knowledge for Front-End Engineers](http://www.amazon.com/High-Performance-Web-Sites-Essential/dp/0596529309) - Steve Souders
  - [Maintainable JavaScript](http://www.amazon.com/Maintainable-JavaScript-Nicholas-C-Zakas/dp/1449327680) - Nicholas C. Zakas
  - [JavaScript Web Applications](http://www.amazon.com/JavaScript-Web-Applications-Alex-MacCaw/dp/144930351X) - Alex MacCaw
  - [Pro JavaScript Techniques](http://www.amazon.com/Pro-JavaScript-Techniques-John-Resig/dp/1590597273) - John Resig
  - [Smashing Node.js: JavaScript Everywhere](http://www.amazon.com/Smashing-Node-js-JavaScript-Everywhere-Magazine/dp/1119962595) - Guillermo Rauch
  - [Secrets of the JavaScript Ninja](http://www.amazon.com/Secrets-JavaScript-Ninja-John-Resig/dp/193398869X) - John Resig and Bear Bibeault
  - [Human JavaScript](http://humanjavascript.com/) - Henrik Joreteg
  - [Superhero.js](http://superherojs.com/) - Kim Joar Bekkelund, Mads Mobæk, & Olav Bjorkoy
  - [JSBooks](http://jsbooks.revolunet.com/)

### Blogs

  - [DailyJS](http://dailyjs.com/)
  - [JavaScript Weekly](http://javascriptweekly.com/)
  - [JavaScript, JavaScript…](http://javascriptweblog.wordpress.com/)
  - [Bocoup Weblog](http://weblog.bocoup.com/)
  - [Adequately Good](http://www.adequatelygood.com/)
  - [NCZOnline](http://www.nczonline.net/)
  - [Perfection Kills](http://perfectionkills.com/)
  - [Ben Alman](http://benalman.com/)
  - [Dmitry Baranovskiy](http://dmitry.baranovskiy.com/)
  - [Dustin Diaz](http://dustindiaz.com/)
  - [nettuts](http://net.tutsplus.com/?s=javascript)


## License

Based on the style guides from [Airbnb](https://github.com/airbnb/javascript) and [bevacqua/js](https://github.com/bevacqua/js)

(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**[[⬆]](#TOC)**
