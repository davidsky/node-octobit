# Octobit (8-bit)
Size, simplicity and performance focused, structured, binary message codec for Nodejs.

## Size and performance
```
// example/index.js
var octobit= require('.././index.js')
var struct= new octobit( require('./example.octo.json') )

// full example
var getQuery= {
      requestId: 35
    , requestType: {get: true, ack: true, noProxy: true}
    , timestamp: Date.now()
    , notInStruct: 'xyz' // ignored as not in structure
    , key: new Buffer('108827d4-e7f0-7d0a-6775-c93236ca00a3')
    , value: new Buffer('some value')
}
var buffer= struct.encode(getQuery)
console.log( buffer.length ) // 64
console.log( struct.decode(buffer).toObject() ) // {requestId: 35, requestDat ... }

// smaller example
var pingQuery= {
	requestId: 12345678
	, requestType: {ping: 1, ack: 1, noProxy: 1, noCache: 1}
}
var buf= struct.encode(pingQuery)
console.log( buf.length ) // 6
console.log( struct.decode(buf).toObject() ) // {requestId: 12345678, requestType: { ping: tru ... }

```
A single core/process on a Mac Air 2012 with i3 CPU can encode 300,000 of the objects per second, and decode (<i>index</i>) all of the resulted buffers in 0.06 second.


## Structures and data types
There are 3 basic data types: `number`, `buffer` and `octet`. Numbers are the usual [U]int32 and double; buffer (byte arrays) is Nodejs Buffer that can be up to 65536 long; and octet is one byte (8 bits) and can hold up to 8 unique flags. __Example structure__ [ key, type, extra ]:
```
// example/example.octo.json
[
	  ["requestId", "uint"]
	, ["requestType", "octet", ["get", "set", "ping", "noCache", "proxy", "noProxy", "faf", "ack"]]
	, ["responseType", "octet", ["get", "set", "error", "proxied", "cached"]]
	, ["timestamp", "double"]
	, ["key", "buffer"]
	, ["value", "buffer"]
]
```
A single `structure` can contain up to `8 elements`. Elements order must not change, but new elements can be appended (up to 8). If elements' order is changed: __data will get scrumbled__.
> when 8 elements is not enough, pack another structure into the first one as a buffer

## encode/decode
* .encode(obj) is a one step process, it directly returns a Buffer that you can flush down a socket.
* .decode(buffer) on the other hand `only reads` the buffer and creates an `index`; it returns an octo-object that allows to get/set specific keys without converting the whole buffer into an object. This allows to proxy data to another server without decoding/re-encoding the whole thing.

> if proxying data is not a concern, use .decode().toObject() to get the complete object at once

## Data types
* 1: `uint` 4 bytes, 32-bit unsigned integer, max value: 4,294,967,295
* 2: `int` 4 bytes, 32-bit signed integer, max value: -/+2,147,483,647
* 3: `double` 8 bytes, 64-bit double-precision floating-point, max value: -/+9,007,199,254,740,991
* 4: `buffer` 2 bytes, byte-array (Buffer), max buffer byte length: 65,535
* 5: `octet` 1 byte, 8 bits, max values (flags): 8

> there's is no `String` support; use `buffer type` like: .encode( { myKey: __new Buffer__('some text') } )

## The `octet` type
The octet type uses 1 byte (8 bits) to store 8 Boolean flags, for example:
```
var octetOnly= struct.encode({requestType: {get: 1, noCache: 1, noProxy: 1}})
console.log(octetOnly) // <Buffer 02 29>
// <29> is the octet (byte) with 3 bits set to 1
console.log( struct.decode(octetOnly).get('requestType') ) // { get: true, noCache: true, noProxy: true }
```
##The format
The format is very simple and can be split into 3 main parts, bits, header and data:
```
+-----+--------+----------------...
[bits]|[header]|[buffers        ...]
+-----+--------+----------------...
```
* __bits__ is an 8 bit index that indicates which of the 8 elements are set
* __header__ contains all the integers, octets, and length of buffers, there are no empty spaces and no offsets, the elements order in the structure is used instead
* __buffers__ contains all the buffers clamped up together at the end of the message

> In the above `octet` example, <i>< Buffer 02 29 ><i> that's 1 byte for message's index to indicate which elements are present ('requestType'), and 1 byte for octet data type to indicate which flags are set ('get', 'noCache' and 'noProxy')

<br>

---

---

# API
## new Octobit(structureArray)
```
var struct= new octobit(structureArray)
```
### octobit.encode(object)
```
var buffer= struct.encode({key: value, key2: value2}) // returns buffer
```
### octobit.decode(buffer)
```
var octObject= struct.decode(buffer) // returns octobject
```

## new Octobject
octo-object is created by the octobit.decode() method:
```
var octObject= struct.decode(buffer) // returns octobject
```
## octobject.get(key)
```
octObject.get() // returns array containing all set keys
octObject.get('key') // value
```
## octobject.set(key, value)
__Work in progress.__
returns true on success
```
octObject.set('requestId', 36) // true
octObject.set('requestType', {noProxy: true}) // true
octObject.set('requestType', 'noProxy') // true
// See TODO
```
## octobject.unset(key[, value])
__Work in progress.__ Only implemented partially.
returns true on success
```
// unset 'noProxy' flag from 'requestType' octet
octObject.unset('requestType', 'noProxy') // true
// See TODO
```
## octobject.getBuffer()
```
octoObject.getBuffer() // <Buffer ... >
```
## octobject.toObject()
```
octoObject.toObject() // returns complete object currently held in the buffer
```


# Installation
```
npm install octobit
```


# TODO
* Octobject.unset(key)
* Octobject.set(key, buffer)
* Octobject.set(newKey, value)


