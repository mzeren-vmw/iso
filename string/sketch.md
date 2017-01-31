# Taxonomy of Strings

## Misc

### Todo
* setup test project.
* tests for static_assert/compilation failure.
* Read: Python 3 strings
* Read: Swift strings
* Read: D-strings.
* Review: Eric T.
* Review: Nicolas Fleury <nidoizo@gmail.com>
* Review: Sofia Mihail, Vlad,


### References
* https://en.wikipedia.org/wiki/UTF-8

## Submission form

### Session Description (Abstract)

At VMware `std::string` has served us well. We have built and shipped large
codebases around it. But that experience has also taught us some of its
shortcommings. Now is a good time to take a step back and re-evaluate the string
design space. Why now? The context in which we write C++ today has changed
dramatically since the dawn of the STL. The UTF family has won the encoding
wars. 64bit addressing will last much longer, in a much wider range of devices,
than prior addressing modes. We have constexper, and soon will have ranges and
reflection. The library now includes `string_view`, string's lighterweight
sidekick, and all three major STL impemenations use the small string
optimization. Even `std::optional`, and smart pointers have some something to
say about strings. In this talk we will disect strings along dimensions such as
storage class, lifetime, encoding, mutability, etc., and then see if we can
put it all back together into a coheasive, modern, set of library and language
features.

### Bio

Mark is an eleven year verteran at VMware where he works on core C++ libraries
and applications for the vSphere product line. Most recently he has been
instigating the move to C++11/14/17 and modernizing VMware's C++ Style Guide. In
2013, during a VMware sponsored sabatical-of-a-lifetime, he worked in the IT
department at SpaceX. In past positions he has worked on iOS application
wrapping, telephony software, and multidimensional databases. When some of our
attendees were still in nappies, he worked in Apple's Evangelism group on
international system software.

### History

This is a new talk, tailored for CppNow, but pertinent to my day job. 2017
SHOULD BE the year that VMware moves from old GCC COW strings to SSO strings and
string_view. Or, maybe, something even better!

### Video

While I occaisionally present technical topics internally at VMware, the last
time I presented publically was back at the 1992 WWDC. Sadly, the Computer
History museum does not have those recordings! However, I have reconstructed a
short internal VMware talk for this submission:

### Comments



## Outline

### Storage Duration
We have the four canonical storage durations:
* **Automatic** - Allocated by the compiler.
* **Static** - Allocated in program image by the compiler and linker.
* **Thread** - Probably not important for this talk.
* **Dynamic** - Allocated dynamically from a runtime library.

#### Automatic strings?
What does it mean for a string to be automatic?
* `std::string` uses dynamic memory so it is not automatic.
  * Except when SSO is in play.
* A static length "short string" would be automatic.
* string_view can be automatic.
* A string constant function argument.

#### More Storage Durations
* **compile time** - e.g. arg to constexpr function. literal exists only at
  compile time. immutable.
* **template values** - theoretically strings could be template **value**
  parameters.

#### The many flavors of Dynamic
* **Interned**
* **special** allocators - smaller indexes shared lifetiem

### Lifetime
* **compiletime**
* **static**
* **automatic** or **unique** or **single owner**
* **shared** in the reference counting way.
* **special** scoped int the life time of the users.

### Linkage
Is this relevant?

### Nullability

Most string implementations can represent null with "zero cost". nullptr for
`const char*`. `std::optional` could take advantage of a null string. Is there
an opportunity for a language extension for a null char[] litteral?

### Mutability

TBD

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

### Data Layout
*Continuity* and *capacity*.

**Continuity**
* **Contiguous**
* **Chunked** - AKA "rope" or "patched"

**Capacity**
* `size_t` - do we really need to have strings this big. Can we use vector or
  array if we do?
* `int` - 32-bit ints leave "half a regiseter" of extra bits on aarch64 and x64.

### Meta strings
* We need strings to be value parameters of templates.

### Type erasure

### Implementation
