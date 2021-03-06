Figson
======

[![Build Status](https://travis-ci.org/declandewet/figson.svg?branch=master)](https://travis-ci.org/declandewet/figson)
[![NPM version](https://badge.fury.io/js/figson.svg)](http://badge.fury.io/js/figson)
[![Dependency Status](https://david-dm.org/declandewet/figson.svg)](https://david-dm.org/declandewet/figson)
[![devDependency Status](https://david-dm.org/declandewet/figson/dev-status.svg)](https://david-dm.org/declandewet/figson#info=devDependencies)
[![Coverage Status](https://coveralls.io/repos/declandewet/figson/badge.png?branch=master)](https://coveralls.io/r/declandewet/figson)

Simple configuration storage.

> **Note:** This project is in early development, and versioning is a little
  different. [Read this](http://markup.im/#q4_cRZ1Q) for more details.

### Why should you care?

This project is in very early development, but already makes working with
JSON, CSON and YAML configuration files a lot easier.

### Installation

```bash
$ npm install figson --save
```

--------------------------------------------------------------------------------

Usage
-----

### Async Example

```javascript
var figson = require('figson');
var path   = require('path');

figson.parse('./config.json', function(error, config) {
  if (error) { throw error; }
  config.set('foo', 'bar');
  config.save(function(error) {
    if (error) { throw error; }
  });
});
```

### Sync Example

```javascript
var figson = require('figson');
var path   = require('path');

try {
  var config = figson.parse('./config.json');
  config.set('foo', 'bar');
  config.save();
} catch (error) {
  throw error;
}
```

You can swap out `config.json` for `config.cson`, `config.yaml` or `config.yml`.
Everything will still work, and uses the exact same API as described below.

### Figson API

Figson itself exposes one method:

#### figson.parse(config_file, [callback]);
Asynchronously reads a configuration file (if there is a callback function), parses it,
and exposes an `error` and a `config` object to the callback (`function(error, config) {}`).
`config_file` is the path to the file. Omit the callback for a synchronous operation.

### Config API

The `config` object is basically just a tiny wrapper around the data inside
the configuration file. It exposes a few properties and methods. All of `config`'s methods
are chainable, and accessing a property with a `config` method uses a tiny
DSL string similar to how you would access that property using JavaScript's dot
notation.

#### config.data
An object representing the configuration file.

#### config.val()
Returns the value of the last key/property in the `config` chain.

#### config.get([key])
Retrieves the value of the given `key`. If no key is provided, it retrieves
the value of the last known key in the chain.

`key` is a string containing the object property who's value you want to retrieve.

Example:

```javascript
// { some: { property: 'value1' }}
config.get('some.property').val() // => value1
// or
config.get().val() // => value1
```

#### config.set([key], value)

Sets the `key` to the `value`. If no `key` is given, uses the most recent key
in the chain. The `value` can be a string, number, object, array or null.

Example:

```javascript
// { an: { array: { property: [] }}}
config.set('an.array.property[0]', 'the value') // { an: { array: { property: ['the value'] }}}
// or
config.set('a different value') // { an: { array: { property: ['a different value'] }}}

```

#### config.update([key], value)

First, attempts a `get()` to determine the existence of the given `key` property. If it
exists, it will then call `set()` with the new `value`. Otherwise, throws an error.
Useful if you need to safely set a value. If no `key` is given, updates the
`key` from the last call in the chain.

Example:

```javascript
config.update('foo', 'baz')
config.get('foo').update('baz')
```

#### config.destroy([key])

"deletes" the given `key` property by setting it's value to `undefined`. If no
key is given, it will "delete" the key from the last method in the chain.

Example:

```javascript
config.destroy('foo');
config.get('bar').destroy()
```

#### config.find(partial_key)

This method is useful for accessing properties that are very deeply nested.
Suppose you have an object:

```javascript
{
  a: {
    very: {
      deeply: {
        nested: {
          property: 'value'
        }
      }
    }
  }
}
```

You want to update `a.very.deeply.nested.property` to have the value `foobar`,
but you don't want to have to type out the whole property name. Just use `.find()`!
As long as the property ends with the key you pass to `find()`, it'll work:

```javascript
config.find('nested.property').set('foobar') // done!
```

#### config.save([callback])

This saves the current state of `config.data` to the configuration file. This is a synchronous
operation, but passing in an optional callback (`function(error) {}`) will make it
perform asynchronously.

Example:

```javascript
config.save(); // synchronous

// asynchronous
config.save(function(error) {
  if (error) { throw error; }
});
```

### Adding your own configuration file types

Figson exposes a small interface for you to add your own configuration file
handlers, if you wish to use a separate data format. Figson uses this interface
internally to add support for JSON, CSON and YAML files.

All you have to do is call `figson.addHandler(name, object)`, where `name` is
the name of your handler (e.g. "CSON" for CSON files) and where `object` is
a JavaScript object that lists file extensions as well as synchronous and
asynchronous parse and stringify operations. For example, if you wanted to
register JSON, you would do this:

```javascript
figson.addHandler('JSON', {
  extensions: ['.json'],
  parse: JSON.parse,
  parseSync: JSON.parse,
  stringify: JSON.stringify,
  stringifySync: JSON.stringify
});
```

Figson will automatically register the file extension to the correct data
format. You can look in
[lib/handlers](https://github.com/declandewet/figson/tree/master/lib/handlers)
for other examples.

Contributing:
-------------

Please read the [contribution guidelines](https://github.com/declandewet/figson/blob/master/contributing.md).
