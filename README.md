# ng-q-plus
# [![NPM version](https://badge.fury.io/js/ng-q-plus.svg)](https://npmjs.org/package/ng-q-plus) [![Build Status](https://travis-ci.org/dbartholomae/ng-q-plus.svg?branch=master)](https://travis-ci.org/dbartholomae/ng-q-plus) [![Dependency Status](https://david-dm.org/dbartholomae/ng-q-plus.svg?theme=shields.io)][https://david-dm.org/dbartholomae/ng-q-plus]

**ng-q-plus** is an angular module enhancing $q promises with additional features

```coffeescript
angular.module 'app', [require('ng-q-plus')]
.service 'myService', ['$q', ($q) ->
  $q.when a: 1
  .get('a')
  .then (value) ->
    console.log value # = 1
]
```

## API

The module can be required via browserify require, as an AMD module via requirejs or as a global, if window.angular is
present. It creates the angular module 'ng-q-plus' and exports its name. This module augments the existing
`$q` service, it does not create a service of its own.
Each promise gets the following new methods:

### `isPending()`
Returns true if the promise is pending (`promise.$$state.status <= 0`)
### `isFulfilled()`
Returns true if the promise is fulfilled (`promise.$$state.status == 1`)
### `isRejected()`
Returns true if the promise is rejected (`promise.$$state.status == 2`)

### `timeout(time_ms, cb)`
Checks after `time_ms` milliseconds whether the promise is still pending. If it is, then `cb` is called with
the deferred object for this promise as an argument. If `cb` isn't a function, the promise instead rejects
with `cb` as error. If `cb` is `undefined`, the promise rejects with `"Timed out after " + time_ms + " ms"`. 
`timeout` is implemented via `$browser.defer`. In tests it has to be flushed via `$browser.defer.flush()`.

### `tap(cb)`
Equivalent to `then (value) -> cb value; return value`

### `get(attr)`
Equivalent to `then (o) -> o[attr]`

### `set(attr, val)`
If `val` and `attr` are not promises this is equivalent to `tap (o) -> o[attr] = val`.
If `val` or `attr` are promises these get resolved and `set` works as written above. 
Please note that this can lead to side effects.

### `post(method, args)`
Equivalent to `then (o) -> o[method].apply o, args`

### `invoke(method, arg1, arg2, ...)`
Equivalent to `then (o) -> o[method].apply o, [arg1, arg2, ...]`

### `send(method, arg1, arg2, ...)`
Alternative name for `invoke`

### `keys()`
Equivalent to `then (o) -> Object.keys o`

### `fapply(args)`
Equivalent to `then (f) -> f.apply undefined, args`

### `fcall(arg1, arg2, ...)`
Equivalent to `then (f) -> f.apply undefined, [arg1, arg2, ...]`

### `all()`
Equivalent to `then (arr) -> $q.all arr`

### `props()`
Alternative name for `all`

### `map(cb)`
For promises for arrays equivalent to `all().then (arr) -> $q.all arr.map cb`.
For promises for objects `obj` the callback gets called as `cb(key, obj[key])`
for all own properties of `obj` and uses the results from the callback to update
 these values.

### `each(cb)`
Equivalent to `@map (el) -> cb el; el`

### `join(p1, p2, ..., cb)`
Calls `cb(v0, v1, v2, ...)` with `v0` being the value of the resolved promise
itself and `v1, v2, ...` being the values of the resolved promises
`v1, v2, ...`. It returns a promise for the result of `cb`.
If any promise rejects, it returns a promise rejected with the same reason. 
