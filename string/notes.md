## Notes


**Capacity**
* `size_t` - do we really need to have strings this big. Can we use vector or
  array if we do?
* `int` - 32-bit ints leave "half a regiseter" of extra bits on aarch64 and x64.

### Meta strings
* We need strings to be value parameters of templates.


### Linkage
Is this relevant?

### Encoding

* **Fixed-length** vs.
* **Variable-length**
* **Codepoints**


Thankfully we now have UTF-8/16/32. It is reasonable to base an entire string
system around these, statically, without making the strings templates.

We are almost always doing the wrong thing for natural language strings since
the standard provides little useful support for variable lenth encodigns.

On the other hand, we often know statically that we will use a fixed length
subset of one of those encodeings. Since we know this statically we can express
this relationship in the type system.

### Null-terminated or not

std::string says mostly that it is not null terminated. Except for c_str. We get
some performance and simplicity by removing c_str.

> **Experiment**: Hack the STL to eliminate null termination in std::string and
> then check effect on generated code. Of course it will not run and is
> incorrect.  Is there some meaning that can be extracted from these numbers?

* Null termination.
* Embedded nulls.

### Reference Layout

SSO strings are big. We need something smaller like shared_string or
unique_string.
### Recap

A string then has a payload and a length:

#### Encoding Summary

We need to model different encodings and conversions between them (ASCII->UTF-8
is free). Each encoding also has a "fixed length / variable length" attribute:

Encoding | Code point size | Element size | Fixed
-------- | :-------------: | :----------: | ----
ASCII    |     1          |    1          | `true`
UTF-8    |    1-4         |    1          | `false`
UTF-16   |     2          |    2          | `true`


.   | In Type | Next to data | Null Term | Next to Reference
--- | --- | --- | --- | ---
Fixed | "small string" | GCC COW | char * | string_view string
Multibyte | | | |

#### Emperical encoding

TODO: Gather data on actual string encoding in memory and at rest from
some of our products. (maybe).
