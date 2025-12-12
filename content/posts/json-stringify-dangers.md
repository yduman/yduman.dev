+++
authors = ["Yadullah Duman"]
title = "The Hidden Dangers of JSON.stringify"
date = "2025-06-24"
description = "A deep dive into JSON.stringify"
categories = ["Node.js"]
tags = ["Node.js"]
+++

In Node.js, it's common practice to log objects during error handling or debugging using `JSON.stringify`. It's one of the first things you're taught when learning Node.js.
Nowadays, I believe you should not use it when debugging, as there are a couple of footguns that are not obvious. Since I encountered this problem while troubleshooting a bug, I wanted to highlight what can happen behind the scenes.

## Serialization Throws

### Circular References

The most common error is probably the famous `Converting circular structure to JSON` error. It can be easily triggered if you try to log request or response objects.

```js
const obj = {}
obj.self = obj
JSON.stringify(obj) // ❌ TypeError: Converting circular structure to JSON
```

### toJSON Can Throw

`toJSON` is a [standardized method in JavaScript](https://tc39.es/ecma262/#sec-serializejsonproperty) that allows objects to define how they should be serialized into JSON.
It works like this:

```js
const user = {
  name: "Alice",
  password: "foobar123",
  toJSON() {
      return { name: this.name }
  }
}

JSON.stringify(user) // '{"name":"Alice"}'
```

Depending on the implementation, this can potentially throw:

```js
const bad = {
  toJSON() {
      throw new Error("oops!")
  }
}

JSON.stringify(bad) // ❌ Uncaught Error: oops!
```

### Getters That Throw

Even if `toJSON` is not defined, `JSON.stringify` will iterate over all **enumerable own properties**. If any are getters that throw, it fails:

```js
const obj = {
  get fail() {
    throw new Error('Boom')
  }
}

JSON.stringify(obj) // ❌ Uncaught Error: Boom
```

This is a lesser-known trap. Non-enumerable getters are ignored:

```js
const obj = {}

Object.defineProperty(obj, 'hidden', {
  get() {
    throw new Error('Ignored')
  },
  enumerable: false
})

JSON.stringify(obj) // '{}'
```

## How JSON.stringify Works

The internal sequence for serialization is as follows:

1. Check for `toJSON` -- If defined, call it and use the result. Exceptions **bubble up**.
2. Property traversal -- Iterate over enumerable own properties. Accessor properties are invoked here. They are invoked, and if any throw, an error is thrown.
3. Structural validation -- Throw a `TypeError` if the resulting object has circular references.

## Handling This the Right Way

Node.js comes with the `util` module, providing [many useful utilities](https://nodejs.org/api/util.html) for us.
\
One of these is the `inspect` function.

`util.inspect` is designed to convert any JavaScript value into a string representation that's readable and useful for debugging, logging and diagnostics. It is available since v0.3.0 and is also used in `console.log`!
It avoids all the pitfalls described above. It bypasses `toJSON`, ignores getters by default and handles circular structures by displaying them instead of throwing. Additionally, you can configure it to inspect deeply nested objects or beautify the output with colors.

```js
const { inspect } = require("node:util")

const circularObj = {}
circularObj.self = circularObj

const toJsonObj = {
  toJSON() {
      throw new Error("oops!")
  }
}

const getterObj = {
  get fail() {
    throw new Error('Boom')
  }
}

inspect(circularObj) // '<ref *1> { self: [Circular *1] }'
inspect(toJsonObj)   // '{ toJSON: [Function: toJSON] }'
inspect(getterObj)   // '{ fail: [Getter] }'
```

## Conclusion

`JSON.stringify` seems like a simple and harmless tool, but as we've seen, it can easily backfire in non-obvious ways. Whether it's circular references, `toJSON` methods, or getters that throw. Relying on it blindly during debugging or error handling can lead to confusing crashes or unexpected behavior.

For everyday logging and inspection, `util.inspect` is the safer and more reliable choice. It gives you control, avoids the pitfalls, and plays nicely with even the most complex objects.

Next time you're troubleshooting something weird, and things break the moment you try to log a response or error — take a closer look. It might just be `JSON.stringify` in the end 😉


