# Section 10. Protocol Buffers Advanced

---

## 57. Integer Types Deep Dive

* Integer Types
  * many ways to represent an integer in protocol buffers:
    * `int32`
    * `int64`
    * `uint32`
    * `uint64`
    * `sint32`
    * `sint64`
    * `fixed32`
    * `fixed64`
    * `sfixed32`
    * `sfixed64`
  * each type is constructed to handle:
    * i. range of allowed values: 64 bits has wider value range than 32 bits
    * ii. whether negative values are allowed: `uint32` vs `sint32`
    * iii. size efficiency on serialisation
  * This is advanced topics for:
    * better performance
    * space optimisation

* Integer Types - Range of allowed values
  * 64 bits allow for a greater range
    * 32 bit:
      * `int32` / `sint32`: -2,147,483,648 to 2,147,483,647
      * `uint32`: 0 to 4,294,967,295
    * 64 bit:
      * `int64` / `sint64`: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
      * `uint64`: 0 to 18,446,744,073,709,551,615

* Integer Types - Negative Values
  * i. `uint32`, `uint64` do NOT allow negative values
  * ii. `int32`, `int64` do NOT encode negative values efficiently
    * negative values constantly use 10 bytes in spaec
  * iii. `sint32`, `sint64` encode negative values well (through the use of a technique called ZigZag)
* choose accordingly based on if your field can have negataive values!

* Integer Types - Size efficiency
  * `uint32`, `uint64`, `int32`, `int64` `sint32`, `sint64` are variable encoding meaning that if they can use less space, they will (for small values)
  * `fixed32` use 4 bytes constantly
    * more efficient than `uint32` if values are often greater than 2^28
  * `fixed64` use 8 bytes constantly
    * more efficient than `uint64` if vaules are often greater than 2^56

---

## 58. Advanced Data Types (`oneof`, `map`, `Timestamp`, and `Duration)

* Advacned Types - `oneof`
  * you can use `oneof` to tell protocol buffers that only one field can have a value:

  ```proto
  message MyMessage {
    int32 id = 1;
    one of example_oneof {
      string my_string = 2;
      bool my_bool = 3;
    }
  }
  ```

  * `oneof` fields **CANNOT** repeated
  * evolving schemas using `oneof` is complicated
    * see documentation if you really need
  * on read, all fields will be `null` except the last one that was set at write

* Advanced Types - `map`s
  * maps can be used to map scalars (except `float` / `double`) to values of any type

  ```proto
  message MyMessage {
    int32 id = 1;
    map<string, Result> results = 2;
  }
  ```

  * map fields cannot be repeated
  * there's no ordering for `map`
    * it's key => value store

* Advanced Types - `Timestamps` (Well Known Types)
  * protocol buffers contain a set of Well Known Types
    * e.g. advanced types known to all programming languages
  * the list is here: [link](https://developers.google.com/protocol-buffers/docs/reference/csharp/class/google/protobuf/well-known-types/timestamp)
  * one of the types is `Timestamp`
    * fields are:
      * `seconds`
      * `nanoseconds` (UTC)
  * do not forget to use `import` statement

  ```proto
  syntax = "proto3";

  import "google/protobuf/timestamp.proto";

  message MyMessage {
    google.protobuf.Timestamp my_field = 1;
  }
  ```

* Advanced Types - `Duration`
  * `Duration` is another Well Known Type
  * It represents the time span between two timestamps
  * It contains, just like `Timestamp`, `seconds` and `nanoseconds`

  ```proto
  syntax = "proto3";

  import "google/protobuf/timestamp.proto";
  import "google/protobuf/duration.proto";

  message MyMessage {
    google.protobuf.Timestamp msg_date = 1;
    google.protobuf.Duration validaty = 2;
  }
  ```

---

## 59. Protocol Buffers Options

* `option`s allow to alter the behaviour of the `protoc` compiler
  * when generating code for specific languages
* there are few bundled options
  * check the docs for details
  * examples:
  
  ```proto
  option csharp_namespace = "Google.Protobuf.WellKnownTypes";
  option cc_enable_arenas = true;
  option go_package = "github.com/golang/protobuf/ptypes/duration";
  option java_package = "com.google.protobuf";
  option java_outer_classname = "DurationProto";
  option java_multiple_files = true;
  option objc_class_prefix = "GPB";
  ```

---

## 60. Naming Conventions

* Naming Convention from the doc
  * Refer to: [HERE](https://developers.google.com/protocol-buffers/docs/style)
  * use **PascalCase** for `message` names
  * use **snake_case** for fields

  ```proto
  message MyMessage {
    string my_long_field = 1;
  }
  ```

  * Use **PascalCase** for `enum`s and **CAPITAL_WITH_UNDERSCORE** for values' names

  ```proto
  enum Foo {
    FIRST_VALUE = 0;
    SECOND_VALUE = 1;
  }
  ```

---

## 61. Uber style guiding

* Uber has an amazing style guide:
  * [github](https://github.com/uber/prototool/blob/dev/etc/style/uber1/uber1.proto)
  * have a look below:

```proto
// Protobuf Uber V1 Style Guide
//
// This is the default style enforced with lint.
//
// There are places in this style guide that reference line places, i.e. the
// first line is always the syntax line, however for demonstration purposes,
// this is violated so we can comment on such style choices here.

// Tab style is two spaces.
// Comments are always //, not /**/.

// The first line is always the syntax line. If there is a license header,
// this comes first, then a newline, then the syntax line.
//
// Syntax should always be "proto3".
syntax = "proto3";

// Next is the package.

package style.uber;

// Next are the package options.
// There is one newline between the package declaration and package options.
// Package options should be alphabetized.

// The below java and golang options are always specified.
// The java options match https://cloud.google.com/apis/design/file_structure

// The go package is always $(basename PACKAGE)pb.
// Do not use the "long-form" package name with a directory path.
option go_package = "uberpb";
// java_multiple_files is always true.
option java_multiple_files = true;
// java_outer_classname is the CamelCased file name without the extension, followed by Proto.
option java_outer_classname = "Uber1Proto";
// The java package is always com.PACKAGE.
option java_package = "com.style.uber";

// Imports come next.
// There is one newline between the package/options and imports.
// Import lines have no newlines between them.
// The imports should be alphabetized.

import "dep/dep.proto";
// Google's well-known types should be directly imported from
// "google/protobuf" as shown.
import "google/protobuf/timestamp.proto";

// Next come the definitions.
// There is one newline between the package options and the definitions.
// The preferred ordering is enums, messages, services.

// IMPORTANT
// 1. All enums have their name as a prefix to all values.
// 2. All enums have a zero value with suffix _INVALID.
// 3. Enums optionally have a one value with suffix _UNSET to denote a
// purposefully unset value, but if you think you will need to denote an
// actually null value over the wire, set this, as _INVALID is not a valid
// value to check against.
// (1) is necessary for C++ scoping rules, (2) is good for golang zero values.
// We prefer _INVALID instead of _NONE or _UNSET as it carries no intention.
// I.e. A programmer would not intentionally set the enum value to _INVALID.
// See FooType as an example.
//
// A longer explanation:
//
// Protocol buffers (v3+ to be specific) does not expose the concept of set vs.
// unset integral fields (of which enums are), as a result it is possible to
// create a empty version of a message and accidentally creating the impression
// that an enum value was set by the caller. This can lead to hard to find bugs
// where 'default' values (the 0 value enum option) is being set without the
// caller knowingly doing so. You may be thinking - but it is super useful to
// just be able to assume my default enum option, just like I want a field
// called count to default to 0 without setting it explicitly. The thing is,
// ENUMs are not integers, they are just represented as them in the proto
// description. Take for example the following enum:
//
// enum Shape {
//     SHAPE_CIRCLE = 0;
//     SHAPE_RECTANGLE = 1;
// }
//
// In this case a consumer of this proto message might forget to set any Shape
// fields that exist, and as a result the default value of 'Circle' will be
// assumed, this is dangerous and creates hard to track down bugs.
//
// The 1 numbered enum should be used for UNSET when this semantic is
// desired {ENUM_TYPE}_UNSET = 1.
//
// Following similar logic to our INVALID case, we don't want information in
// messages to be implied, we want signal to be stated with intention. If you
// have a case where you want UNSET to be a semantic concept then this value
// must be explicitly set. For example:
//
// enum TrafficLight {
//     TRAFFIC_LIGHT_INVALID = 0;
//     TRAFFIC_LIGHT_UNSET = 1;
//     TRAFFIC_LIGHT_GREEN = 2;
//     TRAFFIC_LIGHT_YELLOW = 3;
//     TRAFFIC_LIGHT_RED = 4;
// }
//
// It's tempting to use UNSET as a default value, but then again we risk the
// case of a user forgetting to set the value and it being interpreted as the
// intentional value 'UNSET'. For consistency across our protos if UNSET is
// a semantic value of your enum, it should be field value 1.

// FooType is a foo type, and wants to tell you what type of Foo it is.
enum FooType {
  // There should be no spaces between comments and values.
  // _NONE and _UNSET values should not have comments.

  FOO_TYPE_INVALID = 0;
  FOO_TYPE_UNSET = 1;
  // FOO_TYPE_BALLOON is a balloon that foo likes.
  FOO_TYPE_BALLOON = 2;
  FOO_TYPE_TREE = 3;
}

// Foo is a foo, it goes around the world and talks to bars.
message Foo {
  // If there is a type enum associated with a message, the name of the
  // field should be "type", as is shown here, and it uses tag 1.

  // Type is the type of Foo.
  FooType type = 1;

  // https://developers.google.com/protocol-buffers/docs/proto3#scalar
  // Use the "right" primitive type for the situation, regardless
  // of generated code in the particular target language.
  // For example, use uint32 for ports, not int32, uint64, int64, etc.

  // Note underscore, not CamelCase.

  // https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html

  double why_are_we_using_doubles = 2;

  // See, micros!
  //
  // Here an sint32 is used instead of int32 as this has a high probability
  // of having a negative value, and 32 bits instead of 64 as by definition
  // this will never exceed 32 bits.

  sint32 latitude_micros = 3;
  sint32 longitude_micros = 4;

  // Ids should be strings. Sometimes there are cases where repos have
  // had wrapper message types for ids, but it ends up providing not much
  // if any value in practice, and causes a lot of code uncleanliness when
  // developing.

  string bar_id = 5;
  style.uber.dep.Dep dep = 6;
  google.protobuf.Timestamp timestamp = 7;
}


// Nesting messages and enums is allowed, except for request/response
// messages (see below).
// For example, this appropriate in cases where the inner message
// has no meaning or purpose outside the scope of the outer message.
// NOTE: This affects the names of generated types and may add a great
// amount of verbosity.
// Perform this at your own discretion.

// In this example, Bar has an embedded type, and an ID.

// Bar is a bar that a Foo can talk to.
message Bar {
  // Note the fully-qualified prefix BAR_TYPE_.

  // Type is the type of Bar.
  enum Type {
    BAR_TYPE_INVALID = 0;
    BAR_TYPE_UNSET = 1;
    BAR_TYPE_REMOTE_CONTROL = 2;
    BAR_TYPE_FAN = 3;
  }

  // Type always uses the first tag.

  Type type = 1;

  // An ID field for a message should be the first tag, UNLESS
  // there is a type field, in which case it is the second tag.

  string id = 2;

  // Repeated fields use plural case.
  // Traditionally for Protobuf, repeated fields actually used singular
  // case, but this ends up being more confusing and few actually did this
  // in practice.

  repeated string planet_ids = 3;
}

// Planet is a planet.
message Planet {
  string id = 1;
  string name = 2;
}

// Galaxy is a galaxy.
message Galaxy {
  string id = 1;
  string name = 2;
}

// Quasar is a quasar.
message Quasar {
  string id = 1;
  string name = 2;
}

// ShowingOffBuiltInFieldOptions shows off built in field options.
message ShowingOffBuiltInFieldOptions {

  // Fields that are deprecated should be marked as such.
  //
  // Deprecated is used instead of removing the field and setting the field
  // as reserved because we want to disallow reusing field names, for JSON
  // compatibility. By keeping the field and marking it as deprecated, we
  // make it impossible to reuse either the field tag or field name.
  //
  // Note the space on either side of the =.

  string foo = 1 [deprecated = true];
}

// IMPORTANT: Each individual method needs it's own request and response message,
// even if the message is empty. This for backwards compatibility.
//
// Message names are METHODRequest and METHODResponse.
// This applies to streaming RPC methods as well.
//
// Do not nest enums or other messages in the request or response messages.

message SendFooRequest {
  Foo foo = 1;
}

// Note no newline after opening bracket if the message is empty.

message SendFooResponse {}

message GetPlanetRequest {
  string planet_id = 1;
}

message GetPlanetResponse {
  Planet planet = 1;
}

message StreamGalaxiesRequest {}

message StreamGalaxiesResponse {
  Galaxy galaxy = 1;
}

message StreamQuasarsRequest {
  Quasar quasar = 1;
}

message StreamQuasarsResponse {
  repeated Quasar quasars = 1;
}

// Services should end in Service.
// Use ; to terminate the method definition if there are no RPC options,
// {} if there are RPC options.

// HelloService is the hello service.
service HelloService {
  // Requests and Responses should always match the rpc name.
  // rpc NAME(NAMERequest) returns (NAMEResponse);

  // SendFoo sends a foo.
  rpc SendFoo(SendFooRequest) returns (SendFooResponse);
  // GetPlanet gets a planet.
  rpc GetPlanet(GetPlanetRequest) returns (GetPlanetResponse);
  // StreamGalaxies streams galaxies.
  rpc StreamGalaxies(StreamGalaxiesRequest) returns (stream StreamGalaxiesResponse);
  // StreamQuasars streams quasars.
  rpc StreamQuasars(stream StreamQuasarsRequest) returns (StreamQuasarsResponse);
}
```

---

## 62. Services

* Introduction to Protocol Buffer Services
  * protocol buffers can define `service`s on top of `message`s
  * a `service` is...
    * a **set of endpoints** your application can be accessible from

    ```proto
    service SearchService {
      rpc Search (SearchRequest) returns (SearchResponse);
    }
    ```

  * services need to be interpreted by a framework to generate associated code
  * the main framework is **gRPC**
    * or others...

* Introduction to Protocol Buffer Services
  * gRPC Server <- Protocol Buffer Request <- gRPC Client
  * gRPC Server -> Protocol Buffer Response -> gRPC Client
  * e.g.
    * in a Java Server we have an gRPC server - exposes services (API endpoints)
    * an Go Client which has gRPC Client (Automatically Generated), can send proto request to the gRPC Server inside of the Java server and the go Client (its gRPC client) can get proto response.
    * In the meanwhile, another client, Python client that also has gRPC Client (Augomatically Generated) can do the same thing to the gRPC Server in the Java Server
  * the clients can be automatically generated

* The advantage of Services & RPC (Remote Procedure Calling)
  * you can call your Server API from any client seamlessly
* gRPC for example is used by:
  * Google
  * Netflix
  * CoreOS (etcd)
  * Google Cloud API
  * and gaining popularity fast!

  ```proto
  syntax = "proto3";

  message SearchRequest {
    int32 person_id = 1;
  }

  message SearchResponse {
    int32 person_id = 1;
    string person_name = 2;
  }

  service SearchService {
    rpc Search(SearchRequest) returns (SearchResponse);
  }
  ```

---

## 63. Introduction to gRPC (from gRPC Course)

* Today's trend is to build microservices (MS)
  * MS are built in different lagnuages and encompass a function of your business:
    * in an App
      * Buy MS
      * Delivery MS
      * User MS
      * Promotion MS
      * ...
  * These MS must exchange info. and need to agree on:
    * the API to exchange data
    * the data format
    * the error patterns
    * load balancing
    * many other...
  * one of the popular choice for building API is REST (HTTP-JSON)
    * but we will use **gRPC**

* Building an API is hard
  * need to think about data model:
    * JSON
    * XML
    * something binary?
    * ...
  * need to think about the endpoint
    * `GET /api/v1/user/123/post/456`
    * `POST /api/v1/user/123/post`
  * need to think about how to invoke it and handle errors:
    * API
    * errors
  * need to think about efficiency of the API
    * how much data do I get out of one call?
    * too much data
    * too little data, but many API calls?
  * how about:
    * latency
    * scalability to 1000s of clients?
    * load balancing
    * inter operability with many different languages?
    * authentication
    * monitoring
    * logging
    * ...

* What's an API?
  * at its core, an API is a contract, saying:
    * send me this `REQUEST` (client)
    * I'll send you this `RESPONSE` (server)
  * **it's all about data**
  * the rest, we'll leave to the gRPC framework

* What's gRPC?
  * free and open-source framework
  * developed by google
  * part of the CNCF (Cloud Native Computation Foundation)
    * like: Docker & Kubernetes
  * at a hight level, it allows you to define:
    * `REQUEST`
    * `RESPONSE`
  * for RPC (Remote Procedure Call) and handle all the rest for you.
  * on top of it, it's:
    * modern
    * fast
    * efficient
    * build on top of `HTTP/2`
    * low latency
    * supports streaming
    * language independent
    * super easy to plug in:
      * authentication
      * load balancing
      * logging
      * monitoring

* What's RPC
  * Remote Procedure Call
  * in your CLIENT code, it looks like you're just calling a function directly on the SERVER
    * client code (any language)

    ```rpc
    (code)
    ...
    server.CreateUser(user)
    ...
    (code)
    ```

    * server code (any language)

    ```rpc
    // function creating users
    def CreateUser(User user) {
      ...
    }
    ```

    * RPC call from the client to server over the network
  * it's NOT a new concept (CORBA had this before)
  * with **gRPC**, it's implemented very cleanly and solves a lot of problems
    * e.g. Here are the three nodes
      * i. C++ server (gRPC Server)
      * ii. Ruby Client (gRPC Stub)
      * iii. Android-Java Client (gRPC Stub)
    * between the client and server, they can communicate with:
      * proto request (client to server)
      * proto response (server to client)

* How to get started?
  * at the core of gRPC
    * you need to define the messages and services using **protocol buffers** (`*.proto`)
    * the rest of the gRPC code:
      * automatically generated
      * we just need to provide an implementation for that
    * one `.proto` file works for over 12 programming languages (server and client)
      * allows you to use a framework that scales to millions of RPC per seconds.

* Sneak peak at what we'll do - the `GreetService`

Here is an `example.proto`

```proto
syntax = "proto3";

message Greeting {
  string first_name = 1;
}

message GreetRequest {
  Greeting greeting = 1;
}

message GreetResponse {
  string result = 1;
}

service GreetService {
  rpc Greet(GreetRequest) returns (GreetResponse) {};
}
```

* Why Protocol Buffers?
  * protobufs are language agnostic
  * code can be generated for many languages
  * data is binary
    * efficiently serialised (small payload)
  * very convenient for transporting a lot of data
  * protobufs allow for easy API evolution using rules

before you study gRPC, you should know basic of Protocol Buffers!

* Why should I learn it?
  * many companies have embraced it fully in their production
    * Mercari
    * Google (internally and for Google Cloud services like Pub/Sub)
    * Netflix
    * Square (first contributor, replacement of all their APIs)
    * CoreOS (etcd 3 is built on gRPC for server-server communication)
    * CoackroachDB
  * gRPC
    * future micro-service API
    * mobile-server API
    * (maybe) web APIs

---

## 64. Protocol Buffers Internals

* Protocol Buffers Encoding
  * magic of protobuf
    * to have same serialisation and deserialisation for all the languages
  * serialisation means:
    * transforming an object into bytes
  * deserialisation means:
    * taking bytes and getting an object out of it
  * this part is advanced:
    * only need if you're curious about the internals of Protocol Buffers

* High Level Understanding - Encoding
  * proto schema file
    * e.g.

    * we have a proto file

    ```proto
    message Test {
      string my_string = 1;
    }
    ```

    * and message containing values:
      * go
      * python
      * java

    ```go
    my_string: "testing"
    ```
  * then we get byte streams with:
    * encoding standard rules (done by library we are using)
      * byte streams (e.g. 12 07 74 55 22 6e 11)

* High Level Understanding - Decoding
  * byte streams (e.g. 12 07 74 55 22 6e 11)
  * and
  * proto schema file (e.g. `message test { string my_string = 1;}`)
    * with decoding standard rules (doen by library we are using)
      * we get:
        * message containing values (go, python, java)
          * e.g. `my_string: "testing`

* Decoding Rules for `VarInt`s
  * `VarInt`: a number of variable length when encoded
  * e.g. for number `300`
    * `AC 02`: sequence of 2 bytes - hex form (1 byte is 8 bit)
      * NOTE: `300` is `0xAC02` in `VarInt` form
      * `1010 1100 0000 0010`: sequence of 16 bits
      * `010 1100 0000 0010`: bit sequence after dropping the fist bit (MSB)
        * (if the MSB dropped in 1, read next byte, else stop)
          * `000 0010` ++ `010 1100`: Reverse Order of Byte Sequence
            * `(000 00)10 0101 1100`: concatenate the bits and drop the leading 0s
              * `100101100` = 256 + 32 + 8 + 4 = `300`: read the bit sequence

* Decoding Rules for `message`s
  * `message`s are a series of `<key, value>` pair
  * the key has two components:
    * i. the tag number (`string my_string = 1;`)
    * ii. the "wire" type - takes three bits (the last 3)
  * wire type can take 6 different values:

| **Type** | **Meaning** | **Used for** |
| --- | --- | --- |
| 0 | `VarInt` | `int32`, `int64`, `uint32`, `uint64`, `sint32`, `sint64`, `bool`, `enum` |
| 1 | 64-bit | `fixed64`, `sfixed64`, `double` |
| 2 | length-delimited | `string`, `bytes`, embedded `message`s, packed `repeated` fields |
| 3 | start group | group (deprecated) |
| 4 | end group | group (deprecated) |
| 5 | 32-bit | `fixed32`, `sfixed32`, `float` |

* Decoding Rules for Messages
  * the first field is always a `VarInt` representing the key
  * Schema:
    * `message MyMessage { int32 age = 1; }`
  * Walkthrough on actual Protobuf message: `08 AC 02`
    * `08` = <u>0</u>000 1000 (byte into bits)
    * `000 1000`: read as `VarInt`, drop the MSB(`0`) - first bit
    * take last three bits: `000` = 0
      * wire type `0`, meaning "varint" (see previous)
    * `0001` = 1
      * field tag is `1` (see schema above)
    * key is tag `1`, field type is varint
    * value is `AC 02` (== `300` as we saw)
      * meaning `age = 300` in this instance
  * there are different decoding rules:
    * based on:
      * other wire types
        * see here: [link](https://developers.google.com/protocol-buffers/docs/encoding)
  * overall
    * not really complicated to implement
      * rules are already desinged
      * but very low level (I like it, but some might not)
    * all the libraries you are using respecting this encoding / decoding rules:
      * thus, we get interoperability between all the languages for the generated byte streams
  
---
