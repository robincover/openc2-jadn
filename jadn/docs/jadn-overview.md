# JADN Overview

JSON Abstract Data Notation (JADN) is a language-neutral, platform-neutral,
and format-neutral language for representing information models and for serializing
structured data. [RFC 3444](https://tools.ietf.org/html/rfc3444) discusses the difference
between information models (IMs) and data models (DMs):
* An IM models data objects at a conceptual level independently of any specific implementations or protocols used
to transport the data.
* DMs are intended for implementors and include protocol-specific constructs.

Since conceptual models can be implemented in different ways, multiple DMs can be derived
from a single IM.  An IM can be combined with a set of encoding rules to algorithmically
perform data validation and serialization.  The encoding rules incorporate format-specific
implementation details, reducing or eliminating the need for explicit data models and
freeing implememntors to focus on business logic rather than data handling.

An example data structure is shown below in several formats.  This example,
from the Protocol Buffers documentation, defines a message containing
information about a person:

    message Person {
        required string name = 1;
        required int32 id = 2;
        optional string email = 3;
    }

The Apache Thrift equivalent is:

    struct Person {
        1: string name,
        2: i32 id,
        3: optional string email
    }

A table represenation might look like:
### Person
#### Type: Record

|  ID  |  Name  |  Type  |  #  |Description|
|-----:|--------|--------|----:|---|
|    1 | name   | String |   1 |   |
|    2 | id     | Integer|   1 |   |
|    3 | email  | String | 0..1|   |

This object is represented in JADN as:

    {   "meta": {
            "module": "protobuf-example1"},
        "types": [
            ["Person", "Record", [], "", [
                [1, "name", "String", [], ""],
                [2, "id", "Integer", [], ""],
                [3, "email", "String", ["[0"], ""]]
    ]]}

Although JADN can be edited directly, it is clearer to document data structures
using an interface definition language (IDL) such as Thrift [[1](#ref1)]
or Protobuf [[2](#ref2)].  The IDL or table definitions can be translated
bidirectionally to and from JADN format. One advantage of JADN is that an
IDL parser is not needed in order to use it.  JADN is designed for machine
consumption, and applications can read a JADN schema using nothing but the
standard JSON loader present in most programming languages.

A JADN file consists of meta-information and a list of datatype definitions in
a fixed format.  As shown in the example, each type definition is a list containing
four elements, plus for structured types, a list of field definitions.

### Type definitions
A type definition consists of:
 * name of the type being defined
 * base type from the set of built-in JADN types
 * type options
 * type description
 * field definitions if base type is a structure type

### Field definitions
If the base type is Enumerated, each field definition is a list of
three elements:
 * tag assigned to the item
 * name of the item
 * item description

If the base type is another structure type (Array, Choice, Map, Record),
each field definition is a list of five items:
 * tag or ordinal position of the field
 * name of the field
 * type of the field (built-in or user-defined)
 * field options
 * field description

## Built-in Data Types

A JADN syntax is defined using the following data types:

### Primitive Types

|   Type   | Description |
|---------:|-------------|
Binary     | A sequence of octets or bytes. Serialized either as binary data or as a string using an encoding such as hex or base64.
Boolean    | A logical entity that can have two values: true, and false.  Serialized as either integer or keyword.
Integer    | A number that can be written without a fractional component.  Serialized either as binary data or a text string.
Number     | A real number.  Valid values include integers, rational numbers, and irrational numbers.  Serialized as either binary data or a text string.
Null       | Nothing, used to designate fields with no value.  Serialized as an empty string.
String     | A sequence of characters.  Each character must have a valid Unicode codepoint.

### Structure Types

|   Type   | Description |
|---------:|-------------|
Array      | An ordered list of unnamed fields.  Each field has an ordinal position and a type.  Serialized as a list.
ArraryOf   | An ordered list of unnamed fields of the same type.  Each field has an ordinal position and must be the specified type.  Serialized as a list.
Choice     | One field selected from a set of named fields.  The value has a name and a type. Serialized as a one-element map.
Enumerated | A set of id:name pairs.  Serialized as either the integer id or the name string.
Map        | An unordered set of named fields.  Each field has a name and a type.  Serialized as a mapping type (referred to in various programming languages as: associative array, dict, dictionary, hash, map, object).
Record     | An ordered list of named fields, e.g. a message, record, structure, or row in a table.  Each field has an ordinal position, a name, and a type. Serialized as either a list or a map.

## Options

The Type Options and Field Options items are a list of strings where each string is an option.
The first character is the option ID; the remaining characters are the value.
The option string is converted into a Name: Value pair before use, where the Name corresponds to the ID
and the Value has the type shown in the tables.

### Type Options

|  ID  | ID Char | Name |  Type  |  Description |
|------|:---:|---------|---------|--------------|
  0x3d |  =  | etag    | boolean | enumerated type is serialized as tag, default=false, field name is ignored if present)
  0x5b |  [  | min     | integer | minimum string length, integer value, array length, or property count
  0x5d |  ]  | max     | integer | maximum string length, integer value, array length, or property count
  0x23 |  #  | aetype  | string  | ArrayOf element type
  0x24 |  $  | pattern | string  | regular expression that a string type must match
  0x40 |  @  | format  | string  | name of validation function, e.g., date-time, email, ipaddr, ...

### Field Options
|  ID  | ID Char | Name |  Type  | Description |
|------|:---:|---------|---------|--------------|
  0x5b |  [  | min     | integer | minimum cardinality of field, default = 1, 0 = field is optional
  0x5d |  ]  | max     | integer | maximum cardinality of field, default = 1, 0 = inherited max. If max != 1, field is an array.
  0x26 |  &  | atfield | string  | name of a field that specifies the type of this field
  0x2f |  /  | etype   | string  | serializer-specific encoding type, e.g., u8, i32, hex, base64
  0x21 |  !  | default | string  | default value for this field (coerced to field type)

In the example above, the field options list for the email field [ "[0" ] contains one option.
The option ID is "[" (min) and value is "0", indicating that the minimum cardinality of the
email field is 0, i.e., that field is optional.  

## Serialization
Thrift and Protobuf each define a specific format for serialized data.  JADN is an abstract,
format-independent language for defining datatypes.  Similar to the way HTML separates content
from appearance using CSS, JADN separates data structure from message format using encoding
rules, which allows messages to be serialized using the format most suited to the application
and its operating environment.

For example, a JADN definition of a type called Test1 consisting of a single integer named "a" would be:

    ["Test1", "Record", [], "", [
        [1, "a", "Integer", [], ""]]
    ]

An instance of Test1 with the value a=150 would be serialized
in Protobuf format [[3](#ref3)] as three hex bytes:

    08 96 01

The same instance would be serialized in JSON format as the nine byte string:

    {"a":150}

or in minified JSON format as the five byte string:

    [150]

A codec (encoder/decoder) is an implementation of the encoding rules used to serialize and de-serialize
JADN message instances in a specified format.  Although a JADN schema could be compiled into static
source code for a schema-specific codec, it is designed to be used as "byte-code" interpreted
dynamically by a schema-independent codec library.

## References

1. <a name="ref1">https://thrift.apache.org/docs/idl</a>
2. <a name="ref2">https://developers.google.com/protocol-buffers/</a>
3. <a name="ref3">https://developers.google.com/protocol-buffers/docs/encoding</a> 