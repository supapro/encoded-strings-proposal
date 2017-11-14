# encoded-strings-proposal
C++ Proposal for encoded strings

The goal is to add compile-time information to `std::basic_string` about used encoding along with conversion functions.

## Motivation

It is not uncommon to interact with strings encoded in different character sets in applications of all kinds.
It can be a good idea to provide lightweight instruments with obvious interface with additional extensibility vector to avoid overloading standard library with support of all encoding.

## Proposal

* Add `Encoding` template argument to `std::basic_string` class.
```cpp
template<
  class CharT,
  class Traits = char_traits<CharT>,
  class Allocator = allocator<CharT>,
  class Encoding = string::default_encoding
> class basic_string;
```

* Add template member function `encode` to `std::basic_string`.
```cpp
template<class TargetEncoding>
basic_string<..., TargetEncoding> encode() const;
```
**Note:** `encode` MAY return `std::basic_string` with different `CharT`. This is required to support UTF-16 and similar encodings.

* Add static template member function `decode` to `std::basic_string`.
```cpp
template<class SourceEncoding>
static basic_string<..., TargetEncoding> decode();
```

* Add `std::encoding_convert` function.
```cpp
template<class SourceEncoding, class TargetEncoding>
basic_string<..., TargetEncoding> encoding_convert(const basic_string<..., SourceEncoding>&);
```
**Note:** `encoding_convert` MAY return `std::basic_string` with different `CharT`. This is required to support UTF-16 and similar encodings.

##### Standard Library Encodings

* `std::ascii`
7-bit ASCII encoding.

* `std::native`
System native encoding.

* `std::wide`
System native encoding for wide characters.

* `std::utf8`
UTF-8 (RFC 3629) encoding tag.

* `std::utf16`
UTF-16 (RFC 2781) encoding tag.

* `std::utf32`
UTF-32 encoding tag.

* `std::string::default_encoding`
Implementation-defined encode, can be any of specified above or another unrelated encoding.

##### Encoding Tag
Encoding type used in template arguments is a tag used to pick `std::encoding_convert` implementation.
