# structuredClone polyfill

[![build status](https://github.com/ungap/structured-clone/actions/workflows/node.js.yml/badge.svg)](https://github.com/ungap/structured-clone/actions) [![Coverage Status](https://coveralls.io/repos/github/ungap/structured-clone/badge.svg?branch=main)](https://coveralls.io/github/ungap/structured-clone?branch=main)

An env agnostic serializer and deserializer with recursion ability and types beyond *JSON* from the *HTML* standard itself.

  * [Supported Types](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)
    * *not supported yet*: Blob, File, FileList, ImageBitmap, ImageData, and ArrayBuffer, but typed arrays are supported without major issues, but u/int8, u/int16, and u/int32 are the only safely suppored (right now).
    * *not possible to implement*: the `{transfer: []}` option can be passed but it's completely ignored.
  * [MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)
  * [Serializer](https://html.spec.whatwg.org/multipage/structured-data.html#structuredserializeinternal)
  * [Deserializer](https://html.spec.whatwg.org/multipage/structured-data.html#structureddeserialize)

Serialized values can be safely stringified as *JSON* too, and deserialization resurrect all values, even recursive, or more complex than what *JSON* allows.

### Installation
`yarn add @ungap/structured-clone`

### Examples

Check the [100% test coverage](./test/index.js) to know even more.

EJS example:
```js
// as default export
import structuredClone from '@ungap/structured-clone';
const cloned = structuredClone({any: 'serializable'});

// as independent serializer/deserializer
import {serialize, deserialize} from '@ungap/structured-clone';

// the result can be stringified as JSON without issues
// even if there is recursive data, bigint values,
// typed arrays, and so on
const serialized = serialize({any: 'serializable'});

// the result will be a replica of the original object
const deserialized = deserialize(serialized);
```

CJS example:

```js
// as single funcion
const {default: structuredClone} = require('@ungap/structured-clone');
const cloned = structuredClone({any: 'serializable'});

// as independent serializer/deserializer
const {serialize, serialize} = require('@ungap/structured-clone');

// the result can be stringified as JSON without issues
// even if there is recursive data, bigint values,
// typed arrays, and so on
const serialized = serialize({any: 'serializable'});

// the result will be a replica of the original object
const deserialized = deserialize(serialized);
```

### Extra Features

There is no middle-ground between the structured clone algorithm and JSON:

  * JSON is more relaxed about incompatible values: it just ignores these
  * Structured clone is inflexible regarding incompatible values, yet it makes specialized instances impossible to reconstruct, plus it doesn't offer any helper, such as `toJSON()`, to make serialization possible, or better, with specific cases

This module specialized `serialize` export offers, within the optional extra argument, a **lossy** property to avoid throwing when incompatible types are found down the road (function, symbol, ...), so that it is possible to send with less worrying about thrown errors.

```js
// as default export
import structuredClone from '@ungap/structured-clone';
const cloned = structuredClone(
  {
    method() {
      // ignored, won't be cloned
    },
    special: Symbol('also ignored')
  },
  {
    // avoid throwing
    lossy: true,
    // avoid throwing *and* looks for toJSON
    json: true
  }
);
```

The behavior is the same found in *JSON* when it comes to *Array*, so that unsupported values will result as `null` placeholders instead.

#### toJSON

If `lossy` option is not enough, `json` will actually enforce `lossy` and also check for `toJSON` method when objects are parsed.

Alternative, the `json` exports combines all features:

```js
import {stringify, parse} from '@ungap/structured-clone/json';

parse(stringify({any: 'serializable'}));
```
