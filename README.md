# path-to-regex

> Turn a path string such as `/user/:id` or `/user/:id(\d+)`  into a regular expression 


[![NPM version][npm-image]][npm-url]
[![Dependency Status][david-image]][david-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]

## Installation

```
npm install path-to-regex --save
```

## Usage

```javascript
var pathToRegex = require('path-to-regex');

var matcher = new pathToRegex(path_template, options?);
```

- **path_template** A string or a regular expression.
- **options**
  - **case** When `true` the regexp will be case sensitive. (default: `true`)
  - **splitters** The chars list for splited patch string. (default: `'/'`)
  - **escapeChars** The chars list for escaped. (default: `'/'`)
  - **fromStart** When `true` the regexp will match from the beginning of the string. (default: `true`)
  - **toEnd** When `true` the regexp will match to the end of the string. (default: `true`)

## Samples

#### Demonstration of processing a simple key identifier `:keyname`
```javascript
let parser = new pathToRegex('/foo/:bar');
// parser.regexp:  /^\/foo\/([^\/]+)[\/]?$/

let result = parser.match('/foo/asd');  // result: { bar: 'asd' }

let result = parser.match('/foo/123');  // result: { bar: '123' }

let result = parser.match('/foo/123/bar');  // result: undefined
```




#### Demonstration of processing a key identifier with a specific content `:keyname(\\d+)`
```javascript
let parser = new pathToRegex('/foo/:bar(\\d+)');
// parser.regexp:  /^\/foo\/(\d+)[\/]?$/

let result = parser.match('/foo/123');  // result: { bar: '123' }

let result = parser.match('/foo/asd');  // result: undefined

let result = parser.match('/foo/123asd');  // result: undefined

let result = parser.match('/foo/123/bar');  // result: undefined
```




#### Demonstration of processing a multiple key identifiers `:keyname1 ... :keyname2`
```javascript
let parser = new pathToRegex('/user/:foo/:bar');
// parser.regexp:  /^\/user\/([^\/]+)\/([^\/]+)[\/]?$/

let result = parser.match('/user/123/asd');  // result: { foo: '123', bar: 'asd' }

let result = parser.match('/user/asd/123');  // result: { foo: 'asd', bar: '123' }
```




#### Demonstration of processing a key identifiers with a repeated names `:keyname(\\d+) ... :keyname(\d+)`
```javascript
let parser = new pathToRegex('/foo/:bar/:bar');
// parser.regexp:  /^\/foo\/([^\/]+)\/([^\/]+)[\/]?$/

let result = parser.match('/foo/123/asd');  // result: { bar: [ '123', 'asd' ] }

let result = parser.match('/foo/asd/123');  // result: { bar: [ 'asd', '123' ] }
```




#### Demonstration of processing a key identifier with a quantifier `?`
```javascript
let parser = new pathToRegex('/foo/:bar?');
// parser.regexp:  /^\/foo\/?([^\/]+)?[\/]?$/

let result = parser.match('/foo/123');  // result: { bar: '123' }

let result = parser.match('/foo/');  // result: { bar: undefined }

let result = parser.match('/foo');  // result: { bar: undefined }
```




#### Demonstration of processing a key identifier with a quantifiers `*` and `+`
```javascript
let parser = new pathToRegex('/foo/:bar*');
// parser.regexp:  /^\/foo\/?((?:\/[^\/]+)*)[\/]?$/

let result = parser.match('/foo');  // result: { bar: [] }

let result = parser.match('/foo/');  // result: { bar: [] }

let result = parser.match('/foo/123');  // result: { bar: [ '123' ] }

let result = parser.match('/foo/123/456');  // result: { bar: [ '123', '456' ] }

let result = parser.match('/foo/123/456/');  // result: { bar: [ '123', '456' ] }


let parser = new pathToRegex('/foo/ids: :bar*/:count?');
// parser.regexp:  /^\/foo\/ids\: ?((?:[^\/]+\ ?)*)\/?([^\/]+)?[\/]?$/

let result = parser.match('/foo/ids: 123 456 789');  // result: { bar: [ '123 456 789' ], count: undefined }

let result = parser.match('/foo/ids: 123 456 789/3');  // result: { bar: [ '123 456 789' ], count: '3' }


let parser = new pathToRegex('/foo/:bar+');
// parser.regexp:  /^\/foo\/?((?:\/[^\/]+)+)[\/]?$/

let result = parser.match('/foo');  // result: undefined

let result = parser.match('/foo/');  // result: undefined

let result = parser.match('/foo/123');  // result: { bar: [ '123' ] }

let result = parser.match('/foo/123/456');  // result: { bar: [ '123', '456' ] }

let result = parser.match('/foo/123/456/');  // result: { bar: [ '123', '456' ] }


let parser = new pathToRegex('/foo/ids-,:bar+/:count?');
// parser.regexp:  /^\/foo\/ids-,?((?:[^\/]+\,?)+)\/?([^\/]+)?[\/]?$/

let result = parser.match('/foo/ids-123,456,789');  // result: { bar: [ '123,456,789' ], count: undefined }

let result = parser.match('/foo/ids-123,456,789/3');  // result: { bar: [ '123,456,789' ], count: '3' }
```




#### Demonstration of processing a key identifier with all features
```javascript
let parser = new pathToRegex('/user/:id/bar/:key(\\d+):post?fak/:key(\d+)*:foo+/test/pictures-,:multi(\w+?\.png)*/:key?');
// parser.regexp:  /^\/user\/([^\/]+)\/bar\/(\d+)([^\/]+)?fak\/?((?:\d+\/?)*)?((?:[^\/]+\*?)+)\/test\/pictures-,?((?:\w+?\.png\,?)*)\/?([^\/]+)?[\/]?$/

let result = parser.match('/user/123/bar/111qwertyfak/222foo/test/pictures-p01.png,p02.png,p03.png');
/* result:
 { id: '123',
  key: [ '111', '222' ],
  post: 'qwerty',
  foo: [ 'foo' ],
  multi: [ 'p01.png', 'p02.png', 'p03.png' ] } 
*/

let result = parser.match('/user/123/bar/111qwertyfak/222foo/test/pictures-p01.png,p02.png,p03.png/333');
/* result:
 { id: '123',
  key: [ '111', '222', '333' ],
  post: 'qwerty',
  foo: [ 'foo' ],
  multi: [ 'p01.png', 'p02.png', 'p03.png' ] } 
*/


let parser = new pathToRegex('/user/:id/bar/:key(\\d+):post?fak/:key(\d+)*:foo+/test/pictures- :multi(\w+?\.png)*/:key*');
// parser.regexp:  /^\/user\/([^\/]+)\/bar\/(\d+)([^\/]+)?fak\/?((?:\d+\/?)*)?((?:[^\/]+\*?)+)\/test\/pictures- ?((?:\w+?\.png\ ?)*)\/?((?:\/[^\/]+)*)[\/]?$/

let result = parser.match('/user/123/bar/111fak/222foo/test/pictures-p01.png p02.png p03.png');
/* result:
 { id: '123',
  key: [ '111', '222' ],
  post: undefined,
  foo: [ 'foo' ],
  multi: [ 'p01.png', 'p02.png', 'p03.png' ] } 
*/

let result = parser.match('/user/123/bar/111fak/222foo/test/pictures-p01.png p02.png p03.png/333');
/* result:
 { id: '123',
  key: [ '111', '222', '333' ],
  post: undefined,
  foo: [ 'foo' ],
  multi: [ 'p01.png', 'p02.png', 'p03.png' ] } 
*/

let result = parser.match('/user/123/bar/111fak/222foo/test/pictures-p01.png p02.png p03.png/333/444/');
/* result:
 { id: '123',
  key: [ '111', '222', '333', '444' ],
  post: undefined,
  foo: [ 'foo' ],
  multi: [ 'p01.png', 'p02.png', 'p03.png' ] } 
*/
```



... documentation in processed



[![NPM](https://nodei.co/npm/path-to-regex.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/path-to-regex/)

[npm-image]: https://img.shields.io/npm/v/path-to-regex.svg?style=flat
[npm-url]: https://npmjs.org/package/path-to-regex
[david-image]: http://img.shields.io/david/lastuniverse/path-to-regex.svg?style=flat
[david-url]: https://david-dm.org/lastuniverse/path-to-regex
[license-image]: http://img.shields.io/npm/l/path-to-regex.svg?style=flat
[license-url]: LICENSE
[downloads-image]: http://img.shields.io/npm/dm/path-to-regex.svg?style=flat
[downloads-url]: https://npmjs.org/package/path-to-regex