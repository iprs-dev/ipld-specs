# DAG-PB Spec

**Status: Descriptive - Draft**

DAG-PB does not support the full ["IPLD Data Model."](../../data-model-layer/data-model.md)

## Serial Format

The DAG-PB IPLD serial format is described with a single protobuf:

```protobuf
// An IPFS MerkleDAG Link
message PBLink {

  // binary CID (with no multibase prefix) of the target object
  optional bytes Hash = 1;

  // UTF-8 string name
  optional string Name = 2;

  // cumulative size of target object
  optional uint64 Tsize = 3;
}

// An IPFS MerkleDAG Node
message PBNode {

  // refs to other objects
  repeated PBLink Links = 2;

  // opaque user data
  optional bytes Data = 1;
}
```

The objects link names are specified in the 'Name' field of the PBLink object.
All link names in an object must either be omitted or unique within the object.

DAG-PB aims to have a canonical form for any given set of data. Therefore, in addition to the standard Protobuf parsing rules, DAG-PB decoders should enforce additional constraints to ensure canonical forms (where possible):

1. Fields must appear in their correct order as defined by the Protobuf schema above, blocks with out-of-order fields should be rejected. It is common for Protobuf decoders to accept out-of-order field entries.
2. Duplicate entries in the binary form are invalid, blocks with duplicate field values should be rejected. It is common for Protobuf decoders to accept _updates_ to fields that have already been set.
3. Fields and wire types other than those that appear in the Protobuf schema above are invalid and blocks containing these should be rejected. It is common for Protobuf decoders to skip data in each message type that does not match the fields in the schema.

## Logical Format

When we handle DAG-PB content at the Data Model level, we treat these objects as maps.

This layout can be expressed with [IPLD Schemas](../../schemas/README.md) as:

```ipldsch
type PBNode struct {
  Links [PBLink]
  Data optional Bytes
}

type PBLink struct {
  Hash Link
  Name optional String
  Tsize optional Int
}
```

Constraints:

* The first node in a block of DAG-PB data will match the `PBNode` type.
* `Data` may be omitted or a byte array with a length of zero or more.
* `Links`:
  * must be present, even if empty; the binary form makes no distinction between an empty array and an omitted value, in the Data Model we always instantiate an array.
  * elements must be sorted in ascending order by their `Name` values, which are compared by bytes rather than as strings.
* `Hash`:
  * even though `Hash` is `optional` in the Protobuf encoding, it should not be treated as optional when creating new blocks or decoding existing ones, an omitted `Hash` should be interpreted as a bad block
  * the bytes in the encoding format is interpreted as the bytes of a CID, if the bytes cannot be converted to a CID then it should be treated as a bad block.
  * the data is encoded in the binary form as a byte array, it is therefore possible for a decoder to read a correct binary form but fail to convert a `Hash` to a CID and therefore treat it as a bad block.
* When creating data, you can create maps using the standard Data Model concepts, and as long as they have exactly these fields. If additional fields are present, the DAG-PB codec will error, because there is no way to encode them.

The most recent [JavaScript](https://github.com/ipld/js-dag-pb) implementation strictly exposes this logical format via the Data Model and does not support alternative means of resolving paths via named links as the legacy implementations do (see below). The most recent [Go](https://github.com/ipld/go-ipld-prime-proto) implementation also avoids alternate pathing mechanisms but does not yet support the strict logical format.

## Alternative (Legacy) Pathing

While the [logical format](#logical-format) implicitly describes a set of mechanisms for pathing over and through DAG-PB data in strict Data Model form, legacy implementations afford a means of resolving paths by privileging the `Name` in links.

This alternative pathing is covered here as part of this descriptive spec, but was developed independently of the Data Model and is thus not well standardized.
The alternative pathing mechanisms differ between implementations and has been removed from the newer implementations entirely.

The legacy [Go](https://github.com/ipfs/go-merkledag/tree/master/pb) and [JavaScript](github.com/ipld/js-ipld-dag-pb) implementations both support pathing with link names: `/<name1>/<name2>/…`.

In the legacy Go implementation, this is the only way, which implies that is is impossible to path through nodes that don't name their links. Also neither the Data section nor the Links section/metadata are accessible through paths.

In the legacy JavaScript implementation, there is an additional way to path through the data. It's based purely on the structure of object, i.e. `/Links/<index>/Hash/…`. This way you have direct access to the `Data`, `Links`, and `size` fields, e.g. `/Links/<index>/Hash/Data`.

These two ways of pathing can be combined, so you can access e.g. the `Data` field of a named link via `/<name/Data`. You can also use both approaches within a single path, e.g. `/<name1>/Links/0/Hash/Data` or `/Links/<index>/Hash/<name>/Data`. When using the DAG API in js-ipfs, then the pathing over the structure has precedence, so you won't be able to use named pathing on a named link called `Links`, you would need to use the index of the link instead.

## Canonical DAG-PB

Canonical DAG-PB must:

1. Contain only the specified protobuf fields.
2. Use standard protobuf encoding, with the following field orders:
  - PBNode: Links, Data
  - PBLink: Hash, Name, Tsize

Historical Note: The ordering (Links then Data) of the PBNode message is due to
a bug in the initial protobuf encoder that was used in the first implementation
of ipfs. Take care to maintain this ordering for full compatibility.

## Zero-length blocks

The zero-length DAG-PB block is valid and will be decoded as having null `Data` and null `Links`.

With a SHA2-256 multihash, the CID of this block is:

* CIDv1: bafybeihdwdcefgh4dqkjv67uzcmw7ojee6xedzdetojuzjevtenxquvyku
* CIDv0: QmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n
