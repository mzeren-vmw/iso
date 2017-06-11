## Notes

### COW without COW?

Could I hack libstdc++ to disable COW but keep everything else the same? Make locked always locked.

Or use chap to count reference counts in string blocks?

### string::resize sucks

* ...

### Builders

* What subset of basic_string methods are "builder" methods. If you created
  something that just had those, how frequently could it just "drop in" and
  replace basic_string?


### Uninitialized fill

* Often we want to copy char's while transforming. reserve + push_back is more
  expensive than malloc + transform.

### Nullability

* std::basic_string's data() member cannot return null but
  basic_string_view::data() can.

* zstring/czstring recommendations in the CppCoreGuidelines might be nice for
  completeness.

### Security

* http://www.and.org/vstr/security
* http://www.and.org/ustr/

### Storage class

* Here an interface takes a pointer as part of saying "only give me static
  storage". It's a brittle and error prone mechanism:
  https://reviewboard.eng.vmware.com/r/1110538/diff/2#0.7

* Use a stack based string builder:
  * change 4945037 edit on 2017/03/09 by agesen@wsi (text) 'Avoid some more
    quadratic cost '
  * https://p4web.eng.vmware.com/@md=c&cd=//@/4950297?ac=10
  * Also that does 64bit "string" appends.

### int and long long sized strings

* https://p4web.eng.vmware.com/@md=c&cd=//@/4950297?ac=10
  64bit "string append"

* Petko's fast string comparisons against ints.


### strlen
* `string_view(const char *)` is "narrow" and "Throws: Nothing". Why? Becusa
  strlen comes from C??

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
