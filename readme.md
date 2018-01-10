# fast-safe-stringify

Safe and fast replacement serialization replacement for [JSON.stringify][].

Gracefully handles circular structures instead of throwing.

Provides a stable version as well that will also gracefully handle circular
structures.

## Usage

The same as [JSON.stringify][].

`stringify(value[, replacer[, space]])`

```js
const safeStringify = require('fast-safe-stringify')
const o = { a: 1 }
o.o = o

console.log(safeStringify(o))
// '{"a":1,"o":"[Circular]"}'
console.log(JSON.stringify(o))
// TypeError: Converting circular structure to JSON

function replacer(key, value) {
  console.log('Key:', JSON.stringify(key), 'Value:', JSON.stringify(value))
  // Remove the circular structure
  if (value === '[Circular]') {
    return
  }
  return value
}
const serialized = safeStringify(o, replacer, 2)
// Key: "" Value: {"a":1,"o":"[Circular]"}
// Key: "a" Value: 1
// Key: "o" Value: "[Circular]"
console.log(serialized)
// {
//  "a": 1
// }
```

Using the stable version also works the same as [JSON.stringify][]:

```js
const stableSafeStringify = require('fast-safe-stringify').stable
const o = { b: 1, a: 0 }
o.o = o

console.log(safeStringify(o))
// '{"a":0,"b":1,"o":"[Circular]"}'
console.log(JSON.stringify(o))
// TypeError: Converting circular structure to JSON
```

## JSON.stringify options

[JSON.stringify][]'s `replacer` and `space` options are supported and work the
same as JSON.stringify with one exception: in case a circular structure is
detected the replacer will receive the string `[Circular]` as argument instead
of the circular object itself.

## Benchmarks

Although not JSON, the Node.js `util.inspect` method can be used for similar
purposes (e.g. logging) and also handles circular references.

Here we compare `fast-safe-stringify` with some alternatives:
(Lenovo T450s with a i7-5600U CPU using Node.js 8.9.4)

```md
inspectBench*10000: 57.636ms
jsonStringifySafeBench*10000: 58.737ms
fastSafeStringifyBench*10000: 25.555ms

inspectCircBench*10000: 137.803ms
jsonStringifyCircSafeBench*10000: 110.460ms
fastSafeStringifyCircBench*10000: 38.039ms

inspectDeepBench*10000: 600.103ms
jsonStringifySafeDeepBench*10000: 1345.514ms
fastSafeStringifyDeepBench*10000: 369.198ms

inspectDeepCircBench*10000: 609.102ms
jsonStringifySafeDeepCircBench*10000: 1361.704ms
fastSafeStringifyDeepCircBench*10000: 383.083ms
```

Now we compare `fast-safe-stringify` stable with known alternatives:
(Running the `fast-json-stable-stringify` [benchmark][])

```md
fast-json-stable-stringify x 15,494 ops/sec ±1.59% (88 runs sampled)
json-stable-stringify x 12,229 ops/sec ±1.32% (89 runs sampled)
fast-stable-stringify x 16,226 ops/sec ±0.65% (92 runs sampled)
faster-stable-stringify x 13,900 ops/sec ±1.05% (90 runs sampled)
fast-safe-stringify x 26,528 ops/sec ±1.40% (91 runs sampled)

The fastest is fast-safe-stringify
```

## Protip

Whether `fast-safe-stringify` or alternatives are used: if the use case
consists of deeply nested objects without circular references the following
pattern will give best results.
Shallow or one level nested objects on the other hand will slow down with it.
It is entirely dependant on the use case.

```js
const stringify = require('fast-safe-stringify')

function tryJSONStringify (obj) {
  try { return JSON.stringify(obj) } catch (_) {}
}

const serializedString = tryJSONStringify(deep) || stringify(deep)
```

## Acknowledgements

Sponsored by [nearForm](http://nearform.com)

## License

MIT

[JSON.stringify]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
[benchmark]: https://github.com/epoberezkin/fast-json-stable-stringify/blob/67f688f7441010cfef91a6147280cc501701e83b/benchmark
