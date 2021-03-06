<p align="center">
  <img align="center" src="https://cdn.jsdelivr.net/emojione/assets/svg/1f54e.svg" width="256" height="256" alt="Parse a function">
</p>

# parse-function [![npm version][npmv-img]][npmv-url] [![github release][github-release-img]][github-release-url] [![mit License][license-img]][license-url] [![NPM Downloads Weekly][downloads-weekly-img]][downloads-weekly-url] [![NPM Downloads Total][downloads-total-img]][downloads-total-url] 

> Parse a function into an object using espree, acorn or babylon parsers. Extensible through Smart Plugins

_You might also be interested in [function-arguments][] library if you need more lightweight solution and need for just getting the names of the function arguments._

## Quality Assurance :100:

[![Code Climate][codeclimate-img]][codeclimate-url] 
[![Code Style Standard][standard-img]][standard-url] 
[![Linux Build][travis-img]][travis-url] 
[![Code Coverage][codecov-img]][codecov-url] 
[![Dependencies Status][dependencies-img]][dependencies-url] 
[![Renovate App Status][renovate-img]][renovate-url] 

If you have any _how-to_ kind of questions, please read [Code of Conduct](./CODE_OF_CONDUCT.md) and **join the chat** room or [open an issue][open-issue-url].  
You may also read the [Contributing Guide](./CONTRIBUTING.md). There, beside _"How to contribute?"_, we describe everything **_stated_** by  the badges.

[![tunnckoCore support][gitterchat-img]][gitterchat-url] 
[![Code Format Prettier][prettier-img]][prettier-url] 
[![node security status][nodesecurity-img]][nodesecurity-url] 
[![conventional Commits][ccommits-img]][ccommits-url] 
[![semantic release][semantic-release-img]][semantic-release-url] 
[![Node Version Required][nodeversion-img]][nodeversion-url]

## Features

- **Always up-to-date:** auto-publish new version when new version of dependency is out, [Renovate](https://renovateapp.com)
- **Standard:** using StandardJS, Prettier, SemVer, Semantic Release and conventional commits
- **Smart Plugins:** for extending the core API or the end [Result](#result), see [.use](#use) method and [Plugins Architecture](#plugins-architecture)
- **Extensible:** using plugins for working directly on AST nodes, see the [Plugins Architecture](#plugins-architecture)
- **ES2017 Ready:** by using `.parseExpression` method of the [babylon][] `v7.x` parser
- **Customization:** allows switching the parser, through `options.parse`
- **Support for:** arrow functions, default parameters, generators and async/await functions
- **Stable:** battle-tested in production and against all parsers - [espree][], [acorn][], [babylon][]
- **Tested:** with [275+ tests](./test.js) for _200%_ coverage

## Table of Contents
- [Install](#install)
- [API](#api)
  * [parseFunction](#parsefunction)
  * [.parse](#parse)
  * [.use](#use)
  * [.define](#define)
- [Related](#related)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

This project requires [Node.js][nodeversion-url] v6 and above. Use [npm](https://www.npmjs.com) to install it.

```
$ npm install parse-function
```

## API
Review carefully the provided examples and the working [tests](./test.js).

### [parseFunction](src/index.js#L59)

> Initializes with optional `opts` object which is passed directly
to the desired parser and returns an object
with `.use` and `.parse` methods. The default parse which
is used is [babylon][]'s `.parseExpression` method from `v7`.

**Params**

* `opts` **{Object}**: optional, merged with options passed to `.parse` method    
* `returns` **{Object}**  

**Example**

```jsx
const parseFunction = require('parse-function')

const app = parseFunction({
  ecmaVersion: 2017
})

const fixtureFn = (a, b, c) => {
  a = b + c
  return a + 2
}

const result = app.parse(fixtureFn)
console.log(result)

// see more
console.log(result.name) // => null
console.log(result.isNamed) // => false
console.log(result.isArrow) // => true
console.log(result.isAnonymous) // => true

// array of names of the arguments
console.log(result.args) // => ['a', 'b', 'c']

// comma-separated names of the arguments
console.log(result.params) // => 'a, b, c'
```

### [.parse](src/index.js#L98)

> Parse a given `code` and returns a `result` object
with useful properties - such as `name`, `body` and `args`.
By default it uses Babylon parser, but you can switch it by
passing `options.parse` - for example `options.parse: acorn.parse`.
In the below example will show how to use `acorn` parser, instead
of the default one.

**Params**

* `code` **{Function|String}**: any kind of function or string to be parsed    
* `options` **{Object}**: directly passed to the parser - babylon, acorn, espree    
* `options.parse` **{Function}**: by default `babylon.parseExpression`, all `options` are passed as second argument to that provided function    
* `returns` **{Object}**  

**Example**

```jsx
const acorn = require('acorn')
const parseFn = require('parse-function')
const app = parseFn()

const fn = function foo (bar, baz) { return bar * baz }
const result = app.parse(fn, {
  parse: acorn.parse,
  ecmaVersion: 2017
})

console.log(result.name) // => 'foo'
console.log(result.args) // => ['bar', 'baz']
console.log(result.body) // => ' return bar * baz '
console.log(result.isNamed) // => true
console.log(result.isArrow) // => false
console.log(result.isAnonymous) // => false
console.log(result.isGenerator) // => false
```

### [.use](src/index.js#L164)
> Add a plugin `fn` function for extending the API or working on the AST nodes. The `fn` is immediately invoked and passed with `app` argument which is instance of `parseFunction()` call. That `fn` may return another function that accepts `(node, result)` signature, where `node` is an AST node and `result` is an object which will be returned [result](#result) from the `.parse` method. This retuned function is called on each node only when `.parse` method is called.

_See [Plugins Architecture](#plugins-architecture) section._

**Params**

* `fn` **{Function}**: plugin to be called    
* `returns` **{Object}**  

**Example**

```jsx
// plugin extending the `app`
app.use((app) => {
  app.define(app, 'hello', (place) => `Hello ${place}!`)
})

const hi = app.hello('World')
console.log(hi) // => 'Hello World!'

// or plugin that works on AST nodes
app.use((app) => (node, result) => {
  if (node.type === 'ArrowFunctionExpression') {
    result.thatIsArrow = true
  }
  return result
})

const result = app.parse((a, b) => (a + b + 123))
console.log(result.name) // => null
console.log(result.isArrow) // => true
console.log(result.thatIsArrow) // => true

const result = app.parse(function foo () { return 123 })
console.log(result.name) // => 'foo'
console.log(result.isArrow) // => false
console.log(result.thatIsArrow) // => undefined
```

### [.define](src/index.js#L223)

> Define a non-enumerable property on an object. Just
a convenience mirror of the [define-property][] library,
so check out its docs. Useful to be used in plugins.

**Params**

* `obj` **{Object}**: the object on which to define the property    
* `prop` **{String}**: the name of the property to be defined or modified    
* `val` **{Any}**: the descriptor for the property being defined or modified    
* `returns` **{Object}**  

**Example**

```jsx
const parseFunction = require('parse-function')
const app = parseFunction()

// use it like `define-property` lib
const obj = {}
app.define(obj, 'hi', 'world')
console.log(obj) // => { hi: 'world' }

// or define a custom plugin that adds `.foo` property
// to the end result, returned from `app.parse`
app.use((app) => {
  return (node, result) => {
    // this function is called
    // only when `.parse` is called

    app.define(result, 'foo', 123)

    return result
  }
})

// fixture function to be parsed
const asyncFn = async (qux) => {
  const bar = await Promise.resolve(qux)
  return bar
}

const result = app.parse(asyncFn)

console.log(result.name) // => null
console.log(result.foo) // => 123
console.log(result.args) // => ['qux']

console.log(result.isAsync) // => true
console.log(result.isArrow) // => true
console.log(result.isNamed) // => false
console.log(result.isAnonymous) // => true
```

## Related
- [acorn](https://www.npmjs.com/package/acorn): ECMAScript parser | [homepage](https://github.com/ternjs/acorn "ECMAScript parser")
- [babylon](https://www.npmjs.com/package/babylon): A JavaScript parser | [homepage](https://babeljs.io/ "A JavaScript parser")
- [charlike-cli](https://www.npmjs.com/package/charlike-cli): Command line interface for the [charlike][] project scaffolder. | [homepage](https://github.com/tunnckoCore/charlike-cli#readme "Command line interface for the [charlike][] project scaffolder.")
- [espree](https://www.npmjs.com/package/espree): An Esprima-compatible JavaScript parser built on Acorn | [homepage](https://github.com/eslint/espree "An Esprima-compatible JavaScript parser built on Acorn")
- [hela](https://www.npmjs.com/package/hela): Task runner based on [execa][]. Includes few predefined tasks for linting, testing… [more](https://github.com/tunnckoCore/hela) | [homepage](https://github.com/tunnckoCore/hela "Task runner based on [execa][]. Includes few predefined tasks for linting, testing & releasing")
- [parse-semver](https://www.npmjs.com/package/parse-semver): Parse, normalize and validate given semver shorthand (e.g. gulp@v3.8.10) to object. | [homepage](https://github.com/tunnckocore/parse-semver#readme "Parse, normalize and validate given semver shorthand (e.g. gulp@v3.8.10) to object.")

## Contributing
Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue][open-issue-url].  
Please read the [Contributing Guide](./CONTRIBUTING.md) and [Code of Conduct](./CODE_OF_CONDUCT.md) documents for advices.  

## Author
- [github/tunnckoCore](https://github.com/tunnckoCore)
- [twitter/tunnckoCore](https://twitter.com/tunnckoCore)
- [codementor/tunnckoCore](https://codementor.io/tunnckoCore)

## License
Copyright © 2016-2017, [Charlike Mike Reagent](https://i.am.charlike.online). Released under the [MIT License](LICENSE).

***

_This file was generated by [verb-generate-readme](https://github.com/verbose/verb-generate-readme), v0.6.0, on August 11, 2017._  
Project scaffolded using [charlike-cli][].

[acorn]: https://github.com/ternjs/acorn
[babylon]: https://babeljs.io/
[charlike-cli]: https://github.com/tunnckoCore/charlike-cli
[charlike]: https://github.com/tunnckoCore/charlike
[define-property]: https://github.com/jonschlinkert/define-property
[espree]: https://github.com/eslint/espree
[execa]: https://github.com/sindresorhus/execa
[function-arguments]: https://github.com/tunnckocore/function-arguments

<!-- Heading badges -->
[npmv-url]: https://www.npmjs.com/package/parse-function
[npmv-img]: https://img.shields.io/npm/v/parse-function.svg

[open-issue-url]: https://github.com/tunnckoCore/parse-function/issues/new
[github-release-url]: https://github.com/tunnckoCore/parse-function/releases/latest
[github-release-img]: https://img.shields.io/github/release/tunnckoCore/parse-function.svg

[license-url]: https://github.com/tunnckoCore/parse-function/blob/master/LICENSE
[license-img]: https://img.shields.io/npm/l/parse-function.svg

[downloads-weekly-url]: https://www.npmjs.com/package/parse-function
[downloads-weekly-img]: https://img.shields.io/npm/dw/parse-function.svg

[downloads-total-url]: https://www.npmjs.com/package/parse-function
[downloads-total-img]: https://img.shields.io/npm/dt/parse-function.svg

<!-- Front line badges -->
[codeclimate-url]: https://codeclimate.com/github/tunnckoCore/parse-function
[codeclimate-img]: https://img.shields.io/codeclimate/github/tunnckoCore/parse-function.svg

[standard-url]: https://github.com/standard/standard
[standard-img]: https://img.shields.io/badge/code_style-standard-brightgreen.svg

[travis-url]: https://travis-ci.org/tunnckoCore/parse-function
[travis-img]: https://img.shields.io/travis/tunnckoCore/parse-function/master.svg?label=linux

[codecov-url]: https://codecov.io/gh/tunnckoCore/parse-function
[codecov-img]: https://img.shields.io/codecov/c/github/tunnckoCore/parse-function/master.svg

[dependencies-url]: https://david-dm.org/tunnckoCore/parse-function
[dependencies-img]: https://img.shields.io/david/tunnckoCore/parse-function.svg

[renovate-url]: https://renovateapp.com
[renovate-img]: https://img.shields.io/badge/renovate-enabled-brightgreen.svg

<!-- Second front of badges -->

[gitterchat-url]: https://gitter.im/tunnckoCore/support
[gitterchat-img]: https://img.shields.io/gitter/room/tunnckoCore/support.svg

[prettier-url]: https://github.com/prettier/prettier
[prettier-img]: https://img.shields.io/badge/styled_with-prettier-f952a5.svg

[nodesecurity-url]: https://nodesecurity.io/orgs/tunnckocore-dev/projects/5d75a388-acfe-4668-ad18-e98564e387e1
[nodesecurity-img]: https://nodesecurity.io/orgs/tunnckocore-dev/projects/5d75a388-acfe-4668-ad18-e98564e387e1/badge
<!-- the original color of nsp: 
[nodesec-img]: https://img.shields.io/badge/nsp-no_known_vulns-35a9e0.svg -->

[semantic-release-url]: https://github.com/semantic-release/semantic-release
[semantic-release-img]: https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg

[ccommits-url]: https://conventionalcommits.org/
[ccommits-img]: https://img.shields.io/badge/conventional_commits-1.0.0-yellow.svg

[nodeversion-url]: https://nodejs.org/en/download
[nodeversion-img]: https://img.shields.io/node/v/parse-function.svg

