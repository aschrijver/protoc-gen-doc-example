# Protocol Documentation
<a name="top"/>

## Table of Contents

- [HypercoreSpecV1_md.proto](#HypercoreSpecV1_md.proto)
    - [Cancel](#.Cancel)
    - [Data](#.Data)
    - [Data.Node](#.Data.Node)
    - [Feed](#.Feed)
    - [Handshake](#.Handshake)
    - [Have](#.Have)
    - [Info](#.Info)
    - [Request](#.Request)
    - [Unhave](#.Unhave)
    - [Unwant](#.Unwant)
    - [Want](#.Want)
  
  
  
  

- [Scalar Value Types](#scalar-value-types)



<a name="HypercoreSpecV1_md.proto"/>
<p align="right"><a href="#top">Top</a></p>

## HypercoreSpecV1_md.proto



<a name=".Cancel"/>

### Cancel
Type 8. Cancel a previous Request message that you haven't received yet.

```javascript
message Cancel {
required uint64 index = 1;
optional uint64 bytes = 2;
optional bool hash = 3;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| index | [uint64](#uint64) | required |  |
| bytes | [uint64](#uint64) | optional |  |
| hash | [bool](#bool) | optional |  |






<a name=".Data"/>

### Data
Type 9. Sends a single chunk of data to the other peer. You can send it in response to a
`Request` or unsolicited on it's own as a friendly gift. The data includes all of the
Merkle tree parent nodes needed to verify the hash chain all the way up to the Merkle
roots for this chunk.

Because you can produce the direct parents by hashing the chunk,
only the roots and 'uncle' hashes are included (the siblings to all of the parent nodes).

```javascript
message Data {
message Node {
required uint64 index = 1;
required bytes hash = 2;
required uint64 size = 3;
}
required uint64 index = 1;
optional bytes value = 2;
repeated Node nodes = 3;
optional bytes signature = 4;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| index | [uint64](#uint64) | required |  |
| value | [bytes](#bytes) | optional |  |
| nodes | [.Data.Node](#..Data.Node) | repeated |  |
| signature | [bytes](#bytes) | optional |  |






<a name=".Data.Node"/>

### Data.Node



| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| index | [uint64](#uint64) | required |  |
| hash | [bytes](#bytes) | required |  |
| size | [uint64](#uint64) | required |  |






<a name=".Feed"/>

### Feed
Type 0. Should be the first message sent on a channel.

```javascript
message Feed {
required bytes discoveryKey = 1;
optional bytes nonce = 2;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| discoveryKey | [bytes](#bytes) | required | A BLAKE2b keyed hash of the string 'hypercore' using the public key ofthe metadata register as the key |
| nonce | [bytes](#bytes) | optional | 32 bytes of random binary data, used in our encryption scheme |






<a name=".Handshake"/>

### Handshake
Type 1. Overall connection handshake. Should be sent just after the
Feed message on the first channel only (metadata).

```javascript
message Handshake {
optional bytes id = 1;
optional bool live = 2;
optional bytes userData = 3;
repeated string extensions = 4;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| id | [bytes](#bytes) | optional | 32 byte random data used as a identifier for this peer on the network,useful for checking if you are connected to yourself or another peermore than once |
| live | [bool](#bool) | optional | Whether or not you want to operate in live (continuous) replication modeor end after the initial sync. Both ends must agree to keep theconnection open |
| userData | [bytes](#bytes) | optional | User-specific metadata encoded as a byte sequence |
| extensions | [string](#string) | repeated | List of extensions that are supported on this Feed |






<a name=".Have"/>

### Have
Type 3. How you tell the other peer what chunks of data you have or don't
have.

You should only send Have messages to peers who have expressed interest in
this region by sending Want messages.

```javascript
message Have {
required uint64 start = 1;
optional uint64 length = 2 [default = 1];
optional bytes bitfield = 3;
}
```

When sending bitfields you must run length encode them. The encoded bitfield
is a series of compressed and uncompressed bit sequences. All sequences start with
a header that is a varint.

If the last bit is set in the varint (it is an odd number) then a header represents
a compressed bit sequence.

```javascript
compressed-sequence = varint(
byte-length-of-sequence
<< 2 | bit << 1 | 1
)
```

If the last bit is not set then a header represents an non compressed sequence

```javascript
uncompressed-sequence = varint(
byte-length-of-bitfield << 1 | 0
) + (bitfield)
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | [uint64](#uint64) | required | If you only specify start, it means you are telling the other side youonly have 1 chunk at the position at the value in start |
| length | [uint64](#uint64) | optional | If you specify length, you can describe a range of values that you haveall of, starting from startdefaults to 1 |
| bitfield | [bytes](#bytes) | optional | If you would like to send a range of sparse data about haves/don't havesvia bitfield, relative to start |






<a name=".Info"/>

### Info
Type 2. Message indicating state changes. Used to indicate whether you are
uploading and/or downloading.

Initial state for uploading / downloading is true.
If both ends are not downloading and not live it is safe to consider
the stream ended.

```javascript
message Info {
optional bool uploading = 1;
optional bool downloading = 2;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| uploading | [bool](#bool) | optional |  |
| downloading | [bool](#bool) | optional |  |






<a name=".Request"/>

### Request
Type 7. Request a single chunk of data.

The nodes bitfield is an optional optimization to reduce the amount of duplicate nodes
exchanged during the replication lifecycle. It indicates which parents you have or don't
have. You have a maximum of 64 parents you can specify. Because `uint64` in Protocol Buffers
is implemented as a `varint`, over the wire this does not take up 64 bits in most cases.

The first bit is reserved to signify whether or not you need a signature in response.
The rest of the bits represent whether or not you have (`1`) or don't have (`0`) the
information at this node already.
The ordering is determined by walking parent, sibling up the tree all the way to the root.

```javascript
message Request {
required uint64 index = 1;
optional uint64 bytes = 2;
optional bool hash = 3;
optional uint64 nodes = 4;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| index | [uint64](#uint64) | required | The chunk index for the chunk you want. You should only ask for indexes that you havereceived the `Have` messages for |
| bytes | [uint64](#uint64) | optional | You can also optimistically specify a byte offset, and in the case the remote is able toresolve the chunk for this byte offset depending on their Merkle tree state, they willignore the `index` and send the chunk that resolves for this byte offset instead.But if they cannot resolve the byte request, `index` will be used. |
| hash | [bool](#bool) | optional | If you only want the hash of the chunk and not the chunk data itself |
| nodes | [uint64](#uint64) | optional | A 64 bit long bitfield representing which parent nodes you have |






<a name=".Unhave"/>

### Unhave
Type 4. How you communicate that you deleted or removed a chunk you used to have.

```javascript
message Unhave {
required uint64 start = 1;
optional uint64 length = 2 [default = 1];
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | [uint64](#uint64) | required |  |
| length | [uint64](#uint64) | optional |  |






<a name=".Unwant"/>

### Unwant
Type 6. How you ask to unsubscribe from `Have` messages for a region of chunks from the
other peer. You should only Unwant previously Wanted regions, but if you do `Unwant`
something that hasn't been Wanted it won't have any effect.

The `length` value defaults to `Infinity` or `feed.length` (if not `live`).

```javascript
message Unwant {
required uint64 start = 1;
optional uint64 length = 2;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | [uint64](#uint64) | required |  |
| length | [uint64](#uint64) | optional | Defaults to `Infinity` or `feed.length` (if not `live`) |






<a name=".Want"/>

### Want
Type 5. How you ask the other peer to subscribe you to Have messages for a region of chunks.

'What do we want?' Remote should start sending `Have` messages in this range.

```javascript
message Want {
required uint64 start = 1;
optional uint64 length = 2;
}
```


| Field | Type | Label | Description |
| ----- | ---- | ----- | ----------- |
| start | [uint64](#uint64) | required |  |
| length | [uint64](#uint64) | optional | Defaults to `Infinity` or `feed.length` (if not `live`) |





 <!-- end messages -->

 <!-- end enums -->

 <!-- end HasExtensions -->

 <!-- end services -->



## Scalar Value Types

| .proto Type | Notes | C++ Type | Java Type | Python Type |
| ----------- | ----- | -------- | --------- | ----------- |
| <a name="double" /> double |  | double | double | float |
| <a name="float" /> float |  | float | float | float |
| <a name="int32" /> int32 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32 | int | int |
| <a name="int64" /> int64 | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64 | long | int/long |
| <a name="uint32" /> uint32 | Uses variable-length encoding. | uint32 | int | int/long |
| <a name="uint64" /> uint64 | Uses variable-length encoding. | uint64 | long | int/long |
| <a name="sint32" /> sint32 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32 | int | int |
| <a name="sint64" /> sint64 | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64 | long | int/long |
| <a name="fixed32" /> fixed32 | Always four bytes. More efficient than uint32 if values are often greater than 2^28. | uint32 | int | int |
| <a name="fixed64" /> fixed64 | Always eight bytes. More efficient than uint64 if values are often greater than 2^56. | uint64 | long | int/long |
| <a name="sfixed32" /> sfixed32 | Always four bytes. | int32 | int | int |
| <a name="sfixed64" /> sfixed64 | Always eight bytes. | int64 | long | int/long |
| <a name="bool" /> bool |  | bool | boolean | boolean |
| <a name="string" /> string | A string must always contain UTF-8 encoded or 7-bit ASCII text. | string | String | str/unicode |
| <a name="bytes" /> bytes | May contain any arbitrary sequence of bytes. | string | ByteString | str |

