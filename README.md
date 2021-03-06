# acache

```
npm install acache
```

Sometimes you want to:

1. make an async call to get some data
2. but use a cache if possible

Getting this right requires:

- creating an LRU
- a locking mechanism, so a bunch of calls for the same data on a cold cache don't cause multiple executions of your expensive function for the same data
- an easy uncaching call

This little lib makes it easy.

Coffeescript example using a database call (but really, any expensive async call works).

`ac.query` always calls back with (err, result).

```coffeescript
{ACache} = require 'acache'

ac = new ACache {maxAgeMs: 10000, maxStorage: 100}

# pick a string for the cache key:

key = "#{uid}-#{friend_uid}"

# say, get something from the database, given 2 uid's
ac.query {
  keyBy: key
  fn: (cb) ->
    mysql.query 'SELECT BLEAH WHERE FOO=? AND BAR=?', [uid, friend_uid], (err, rows, info) ->
      res = {rows, info}
      cb err, res
}, defer err, res

# remove something from the cache
ac.uncache {keyBy: key}

# manually add something
ac.put {keyBy: key}, val

# check some basic stats
console.log ac.stats() # size, hits, misses, etc.
```

### Curious if the cache was hit

```coffeescript
It calls back with a 3rd boolean param, whether the cache was hit:
ac.query {
  keyBy: whatever
  fn: (cb) -> cb null, "hello world"
}, defer err, res, did_hit
```

### constructor params:

- `maxAgeMs`: max time to store something in cache
- `maxStorage`: max answers to cache

### `query` params:

- arg0 (object):
  - `keyBy` : a key for this cache call. Feel free to pass a string, number, or object; it will be hashed
  - `fn` : a function to run, to fill the cache, if it's missing. your function should take one parameter, `cb`. It should then call `cb` with `err, res`
- arg1 (fn) :
  - a function you want called with the results of your `fn`. Say, `err, res` from either the cache or hot read

## Errors

This does not cache errors. If you want to cache errors you can easily call back with `null, {err, res}`.
