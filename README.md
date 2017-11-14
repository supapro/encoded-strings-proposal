# encoded-strings-proposal
C++ Proposal for encoded strings

* Document number: PxxxxRy
* Maks Mazurov \<foxcpp@yandex.ru\>
* Target audience: LEWG, LWG

## Table of contents
* [Introduction](#introduction)
* [Motivation](#motivation)
* [Proposal](#proposal)
  * [Standard Library Encodings](#standard-library-encodings)
  * [Encoding Tag](#encoding-tag)

## Introduction

The goal is to add compile-time information to string classes about used encoding along with conversion functions.

## Motivation

* Provide more flexible replacement for deprecated `std::wstring_convert`.
* You already have `std::string` and/or `std::string_view` all around your code. Why copy to 3rd-party's encoding-aware string, when you can operate on standard string?
* ...

## Proposal

**Note:** Declarations written as if they are inside std namespace (`std::` is omitted). `...` is all other template arguments.

* Add `Encoding` template argument to `std::basic_string` and `std::basic_string_view`.
```cpp
template<
  class CharT,
  class Traits = char_traits<CharT>,
  class Allocator = allocator<CharT>,
  class Encoding = string::default_encoding
> class basic_string;

template<
  class CharT,
  class Traits = char_traits<CharT>,
  class Encoding = string_view::default_encoding
> class basic_string_view;
```

* Add template member function `to_encoding` to `std::basic_string` and `std::basic_string_view`.
```cpp
template<class TargetEncoding>
basic_string<..., TargetEncoding> to_encoding() const;
```
**Note:** `to_encoding` may return `std::basic_string` with different `CharT`. This is required to support UTF-16 and similar encodings.


#### Standard Library Encodings

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
Implementation-defined encoding, can be any of specified above or another unrelated encoding.
Not required to be same between program runs.

* `std::string_view::default_encoding`
Must be same as `std::string::default_encoding`.

Implementation __can__ provide additional encodings.

#### Encoding type

The implementation is allowed to store some information about encoding in static fields.

Example:
  We have encoding conversion library that identifies various encoding using string names.
  It is allowed to use encoding tag static field (preferably constexpr) named `encoding_name`.
  
  ```cpp
  struct win1251 {
    static constexpr const char* library_encoding_name = "cp1251";
    
    template<class TargetEncoding>
    static basic_string<..., TargetEncoding> to_encoding(const char* sptr, size_t length) {
      library_convert(TargetEncoding::library_encoding_name, win1251::library_encoding_name);
    }
  };
  ```
