# async-json

  async-json is a library that provides an asynchronous version of the standard JSON.stringify.

  This can be used in both the browser and in node.js. The only dependency is that `JSON` must exist either by the
  implementation providing it (all modern browsers and implementations) or by providing it yourself
  [json2.js](https://github.com/douglascrockford/JSON-js).

## Installation

  Assuming you are using node.js:

  The easiest way to install is through [npm](http://npmjs.org/).

    $ npm install async-json

  Alternatively, you can pull from [github](https://github.com/ckknight/async-json) and place where necessary.

## API

  There is only one, very simple function: stringify.

  It takes a javascript object as its primary argument, which is the object to stringify.

  Then if you prefer to use ES6 Promises, which are supported in many engines (but not all), and have many great polyfills (e.g. [es6-promise](https://github.com/jakearchibald/es6-promise)), then the API is as such:

    var asyncJSON = require('async-json');

    asyncJSON.stringify({ some: "data" })
      .then(function (jsonValue) {
        assert(jsonValue === '{"some":"data"}');
      });

  Otherwise, if you prefer node.js-style callbacks, then you can pass a callback with two parameters:

  * The first is the error that occurred or `null` if there is no error.
  * The second is the result JSON value.

    var asyncJSON = require('async-json');

    asyncJSON.stringify({ some: "data" }, function (err, jsonValue) {
        if (err) {
            return handleError(err);
        }

        assert(jsonValue === '{"some":"data"}');
    });

  The callback is guaranteed to fire no earlier than the next tick, so any synchronous code after calling `.stringify` will execute before the callback.

  If a large object or array is passed in, the stringifier may occasionally pause a tick so the execution thread isn't continuously blocked.

  In the first argument, if a function is found, rather than skipping it as `JSON` does, it is invoked.

  Two types of functions can be provided:

  * A synchronous function, of which the return value is used to replace the function value in output. This is useful
    for lazy values.
  * An asynchronous function, which takes a callback. This is useful for I/O-related values.

### Synchronous/Lazy functions

  Synchronous functions are useful for handling a value on-the-fly that you do not wish to precompute.

    var asyncJSON = require('async-json');

    asyncJSON.stringify({ some: function() { return "hard to compute data"; } }, function (err, jsonValue) {
        if (err) {
            throw err;
        }

        jsonValue === '{"some":"hard to compute data"}';
    });

  Like the previous example, this is invoked immediately, and the function will be invoked along the pipeline. Since
  JavaScript lacks the concept of background threads, this may cause the program to freeze while the value is being
  computed.

  Functions will be invoked with their `this` context as the object they are part of, just as a normal method.

  You can also return Promises (or any thenable) from your functions which will be used to try to resolve the eventual value to then serialize.

      var asyncJSON = require('async-json');

      asyncJSON.stringify({ some: function() {
        return Promise.resolve("hard to compute data");
      } }, function (err, jsonValue) {
          if (err) {
              throw err;
          }

          jsonValue === '{"some":"hard to compute data"}';
      });

  *Note: it is perfectly acceptable to return an object that contains another function, it will be calculated through
  the same process.*

## Asynchronous functions

  Asynchronous functions are useful for handling I/O driven data or if you have a gradual calculation mechanism.

    var asyncJSON = require('async-json');

    asyncJSON.stringify({ data: function(callback) {
        SomeDatabase.find({}, function (err, value) {
            if (err) {
                callback(err);
            } else {
                callback(null, {
                    key: value.someNumber
                });
            }
        });
    } }, function (err, jsonValue) {
        if (err) {
            throw err;
        }

        jsonValue === '{"data":{"key":12345}}';
    });

  This fetches data from some database or cache or other external source, and once it has a resultant value or error,
  will call the async callback function.

    var asyncJSON = require('async-json');

    asyncJSON.stringify({ data: function(callback) {
        setTimeout(function () {
            callback(null, "Hello there!");
        }, 1000);
    } }, function (err, jsonValue) {
        if (err) {
            throw err;
        }

        jsonValue === '"Hello there!"';
    });

  This will invoke the async callback one second after calling `stringify` with the obvious result.

  Promises (or thenables) can also be provided to the callback, which will be appropriately resolved.

## Running Tests

first:

    $ git submodule update --init

then:

    $ make test

## Issues

  If you find any issues with async-json or have any suggestions or feedback, please feel free to visit the [github
  issues](https://github.com/ckknight/async-json/issues) page.

## License

MIT licensed. See [LICENSE](https://github.com/ckknight/async-json/blob/master/LICENSE) for more details.
