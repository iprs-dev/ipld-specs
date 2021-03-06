# Specification: IPLD Data Model

**Status: Descriptive - Draft**

The IPLD Data Model is a core part of the IPLD specification,
which describes what data is representable in IPLD --
for example, booleans, integers, textual strings, maps and lists, etc.

While the Data Model describes these representations in the abstract,
[Codecs](../block-layer/codecs) specify exactly how these data are transcribed
into serialized bytes. (Another component of the IPLD specifications,
[Schemas](../schemas), provide additional optional tooling on top of the Data
Model which can further refine, describe, and constrain the range of acceptable
data values.)

Motivation
----------

There is not **one** block format but **many** block formats widely used today in content 
addressed data structures. We assume that we'll see more of these block formats in the 
future and not less. It is quite clear then that a reasonable and more future proof approach 
to using these data structures is to be block format agnostic.

The data model defines a common respresentation of basic types that **are easily representable
by common programming languages.** This provides the foundation for block format agnostic tools
to be built using familiar native types in a programmer's preferred language. As such, there
is an element of "lowest common denominator" to the IPLD Data Model in that it cannot support
some advanced features (like non-string keys for Maps) because support for such a feature
is not common enough among programming languages.

This does not mean that a block format could not support more advanced features than exist in the 
data model, it just means that the common set of tools IPLD is building w/ its block format 
agnostic approach cannot be easily leveraged to use those features.

Kinds
-----

The following is the list of essential _kinds_ (or more formally, _representation kinds_)
of data representable in the IPLD Data Model:

* Null
* Boolean
* Integer
* Float
* String
* Bytes
* List
* Map
* Link

(Note that we use the term "_kinds_" here to disambiguate this from "_types_",
which is a term we'll use at the [Schemas](../schemas) level.)

The _recursive kinds_ are:

* List
* Map

The _scalar kinds_ (the complement of recursive) are:

* Null
* Boolean
* Integer
* Float
* String
* Bytes
* Link

(Note that [Schemas](../schemas) introduce a few more kinds -- when clarification is necessary,
these Data Model kinds can be called the "_representation kinds_",
while the additional kinds introduced in the Schema layer are "_perceived kinds_".)

### Kinds Reference

Each of the following sections provides details about each of the kinds
introduced in the summary table above.

#### Null kind

Null is a scalar kind.  Its cardinality is one -- the only value is 'null'.

#### Boolean kind

Boolean is a scalar kind.  Its cardinality is two -- either the value 'true' or the value 'false'.

#### Integer kind


#### Float kind


#### String kind

String is a scalar kind.  Its cardinality is infinite -- strings do not have a length limit.

Strings should be UTF-8 text, in [NFC](https://www.unicode.org/reports/tr15/#Norm_Forms) canonicalization.

#### Bytes kind

Bytes is a scalar kind.  Its cardinality is infinite -- byte sequences do not have a length limit.

Bytes are distinct from strings in that they are not considered to have any character encoding nor
generally expected to be printable as human-readable text.
In order to print byte sequences as text, additional effort such as Base64 encoding may be required.

#### List kind

List is a recursive kind.

Values contained in lists can be accessed by their ordinal offset in the list.

#### Map kind

Map is a recursive kind.

Values in maps are accessed by their "key".  Maps can also be iterated over,
yielding key+value pairs.

#### Link kind

A link represents a link to another IPLD Block. The link reference
is a [`CID`](CID.md).

Link is a scalar kind -- however, when "loaded", may become another kind, either scalar or recursive!

### Kinds Implementation References

- Kinds in Go: https://github.com/ipld/go-ipld-prime/blob/master/kind.go
