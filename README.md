# Plasma v4

Implementation of [node-organic/Plasma v1.0.0](https://github.com/VarnaLab/node-organic/blob/master/docs/Plasma.md).

## Public API

### plasma.emit(c [, callback]) : Promise<Array>

Immediatelly triggers any reactions matching given `c` chemical.

___arguments___
* `c` - Emitted chemical
  * as `String` equals to `{ type: String, ... }` Chemical
  * as `Object` equals to Chemical

___returns___

* `Promise<Array>` - returns a Promise which resolves to Array of Objects 
been the result of any reactions triggered by the emit.

___throws___

* `Array<Error>` - throws an Array of Error objects

___notes___

* Awaits any reactions in sequence based on their registration via `plasma.on` 
* Halts when a reaction throws
* Doesn't halt when a reaction returns error via callback signature

### plasma.emitOnce(c) : Promise

Does the same as `plasma.emit` but triggers only the first matched
reaction.

___returns___

* `Promise` - returns a Promise which resolves to the result of the reaction matched by the emit pattern.

### plasma.store(c) : Promise<Array>

Does the same as `plasma.emit` but also triggers any
reactions registered in the future using `plasma.on`

This method actually stored inmemory the `c` chemical referenced within the plasma as `storedChemical`.

___returns___

* `Promise<Array>` - returns a Promise which resolves to Array of Objects 
been the result of any reactions been triggered by the emit.

### plasma.storeAndOverride(c) : Promise<Array>

Does the same as `plasma.emit` but also triggers any
reactions registered in the future using `plasma.on`.

It overrides previously stored chemicals having the same chemical using `c.type`

___returns___

* `Promise<Array>` - returns a Promise which resolves to Array of Objects 
been the result of any reactions been triggered by the emit.

### plasma.has(pattern) : boolean

Checks synchronously for stored chemicals by given `pattern`.

___returns___

* `Boolean` - returns `true` when plasma contains a stored chemical matching given `pattern`

### plasma.get(pattern) : Chemical

Returns synchronously first found stored chemical by given pattern.

### plasma.getAll(pattern) : Array [ Chemical ]

Returns synchronously all stored chemicals by given pattern.

### plasma.on(pattern, function (c [, callback]){} [, context])

Registers a function to be triggered when chemical emitted in plasma matches given pattern.

___arguments___
* `pattern` - matches emitted chemicals
  * as `String` matching `Chemical.type` property
  * as `Object` matching one or many properties of `Chemical`
* `c` - `Object` Chemical matching `pattern`
* `callback` - *optional* callback function used for feedback
* `context` - *optional* context to be used for calling the function

___example___

```
plasma.on({type: 'myChemical', kind: 'v1'}, async function (c) {
  console.log(c) // {type: 'myChemical', kind: 'v1', ...}
  await operation()
  return result
})

let result = await plasma.emit({
  type: 'myChemical',
  kind: 'v1',
  property: 'value'
})
```

### plasma.once(pattern, function (c [, callback]){} [, context])

The same as `plasma.on(pattern, function reaction (c){})` but will trigger the function only once.

### plasma.on([p1, p2, ...], function (c1, c2, ...){} [, context])

Registers a function to be triggered when all chemicals emitted in plasma have been matched.

___arguments___
* `p` - array
  * having elements `String` matching `Chemical.type` property
  * having elements `Object` matching one or many properties of `Chemical`
* `c` - array
  * `Object` Chemicals matching `p` array maintaining their index order
* `context` - *optional* context to be used for calling the function

### plasma.once([p1, p2, ...], function (c1, c2, ...) {} [, context])

The same as `plasma.on([p1, p2], function(c1, c2){})` but will trigger the function only once.

### plasma.off(pattern, function, context)

Unregisters chemical reaction functions, the opposite of `plasma.on` or `plasma.once`.

___arguments___

* `pattern`
  * as `String` or `Array` or `Object` - needs function handler for unregister to succeed
  * as `Function` - finds **all** registered chemical reactions with that function handler and unregisters them.
* `function` *optional* required only when `pattern` is String, Array or Object. This should be the exact function used for `plasma.on` or `plasma.once`
* `context` *optional* used to scope removing of chemical reactions within context

### plasma.off(function, context)

Unregisters chemical reaction functions, the opposite of `plasma.on` or `plasma.once`.

___arguments___

* `function` this should be the exact function used for `plasma.on` or `plasma.once`
* `context` *optional* used to scope removing of chemical reactions within context

### plasma.trash(c)

Removes previously stored chemical via `plasma.store`. It does removal by reference and won't throw exception if given chemical is not found in plasma's store.

### plasma.trashAll(pattern)

Removes previously stored chemicals via `plasma.store`. It does removal by chemical pattern

### plasma.pipe(function(c){})

Method which will invoke function per any chemical been emitted or stored in plasma.

### plasma.unpipe(function(c){})

Stops invoking given function previously used for `plasma.pipe`

## Features

Current implementation of organic plasma interface has the following addon features designed for ease in daily development process.

### optional usage of arguments

Invoking **either**:

* `plasma.emit('ready')`
* `plasma.emit({type: 'ready'})`

triggers **all** the following:

* `plasma.on('ready', function(c){ c.type === 'ready' })`
* `plasma.on({type: 'ready'}, function(c){ c.type === 'ready' })`

### feedback support


#### using callback

```
plasma.on('c1', function reaction1 (c, callback) {
  callback(c)
})
plasma.emit('c1', function callback (c) {
  // c.type === 'c1'
})
```

#### using async/await

```
plasma.on('c1', async function reaction1 (c) {
  let result
  return result
})
let result = await plasma.emit('c1')
console.log(result)
```

### custom pattern <-> chemical match algorithms

* override `plasma.utils.deepEqual(pattern, chemical)`

### match chemicals using class definitions as pattern

```
var Class1 = function () {}
plasma.on(Class1, function (instance) {

})
var instance = new Class1()
plasma.emit(instance)
```

### throw on missing listeners/handlers for a chemical

When emitting a chemical (via the `.emit` method) which has no registered handlers (via the `.on`/`.once` methods) an Error will be thrown

* The missing handlers error chemical will not be thrown when storing chemicals.
* by default this feature is disabled unless `throwOnMissingHandler` value is set to `true`

```
var instance  = new Plasma({
  throwOnMissingHandler: true
})

instance.emit("chemicalWithNoHandler") // throws Error
```