# Strings Sketch

## Misc

### Todo
* setup test project.
* tests for static_assert/compilation failure.
* Read: Python 3 strings
* Read: Swift strings
* Read: D-strings.
* Watch: Nicolas Fleury CppCon 16.
* Review: Eric T.
* Review: Nicolas Fleury <nidoizo@gmail.com>
* Review: Sofia Mihail, Vlad,


### References
* https://en.wikipedia.org/wiki/UTF-8
* `text_view`
  * http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0244r1.html
* `folly::FixedString`
  * https://github.com/facebook/folly/blob/master/folly/FixedString.h
* `std::codecvt`
  * http://en.cppreference.com/w/cpp/locale/codecvt

## Submission form

### Title

Rethinking Strings

### Session Description (Abstract)

At VMware `std::string` has served us well. We have built and shipped large
codebases around it. However, that experience has also taught us some of its
shortcomings. Now is a good time to take a step back and re-evaluate the string
design space. Why now? The context has changed dramatically since the dawn of
`basic_string`. The UTF family is the de facto encoding standard. 64bit
addressing will last much longer, in a much wider range of devices, than prior
addressing modes.  We have `constexpr`, and soon will have ranges and
reflection. The library now includes `string_view`, all three major STL
implementations use the small string optimization, and Eric Neibler has given us
`folly::FixedString`. A `std::text_view` proposal is in flight. Even
`std::optional`, and smart pointers have some something to say about strings.

In this talk we will dissect strings along dimensions such as storage duration,
encoding, mutability, etc., and then see if we can knit it all back together
into a cohesive, modern, set of library and language features. I won't have all
the answers, so show up and speak up!

### Bio

Mark is an eleven-year veteran at VMware where he works on core C++ libraries
and applications for the vSphere product line. Most recently he has been
instigating the move to C++11/14/17 and modernizing VMware's C++ Style Guide. In
2013, during a VMware sponsored sabbatical-of-a-lifetime, he was embedded in the
IT department at SpaceX. In past positions, he has worked on iOS application
wrapping, software telephony, and multidimensional databases. When some of our
attendees were still in nappies, he worked in Apple's Evangelism group on
international system software.

### History

This is a new talk, tailored for CppNow.

### Video

While I occasionally present technical topics internally at VMware, the last
time I presented publicly was back at the 1992 WWDC. Sadly, the Computer
History museum does not have those recordings! However, I have reconstructed a
short internal VMware talk for this submission:

### Audience Description

Library writers, performance critical systems, l10n experts

### Comments


## Outline

### Intro

... I will start with two thoughts that motivated this talk...

#### 1. The COW is Dead

I work with code that still uses GCC's copy on write (COW) strings. We have even
*tuned* our code to that implementation. However, we know that we must move to a
standard compliant implementation sooner rather than later. I think about this
transition a lot. I see it as an opportunity to re-think our use of strings.
Building this talk is part of that process.

TODO: Extend this section with an explanation of why C++11 disallows COW?

#### 2. `string_view` is a Beautiful Thing...

... almost.

GCC's small string optimized (SSO) strings have a layout like this:

```c++
struct sso_string {
  char *data;
  size_t len;
  char extra[16];
};
```

When I first saw that I thought "of course" because then clearly a
`string_view` is just:

```c++
struct string_view {
  char *data;
  size_t len;
};
```

So, a string slices effortlessly to a string_view. Brilliant!

Then I looked at GCC's `string_view` and found:

```c++
struct string_view {
  size_t len;
  char* data;
};
```

Sigh. Well at least changing one or the other and measuring the effects would be
a nice experiment!

TODO: check libc++, msvc.

#### Composing and Converting

This got me thinking: How could string-ish things compose and convert nicely
and more generally?

Consider:
```c++
struct c_string {
  char *data; // null terminated.
};
```

Thus an `sso_string` IsA `string_view` *or* IsA `c_string` (as long as
the `sso_string` null terminates data).

What if we stole some bits from the size? On 64-bit systems, `size_t` seem like
overkill ,so we have a lot of room to maneuver.

### What If ..

I realized that this was an opportunity to re-think things more
broadly; to see if we could address many issues in one go. Were there more
"effortless conversions" that could be provided? What other use cases could we
cover with those extra bits?

So, I dusted off a long "wish list" for strings that we had accumulated over the
years. Let's review some of the highlights.

### History

#### std::string Everywhere

The codebase that I work on began over ten years ago with a clean slate and
adopted "std::string everywhere". This worked well while the project was
evolving quickly and adding features. Later as changes slowed and we started
focusing on macro-algorithmic performance, it still worked.

However, several years ago we started working more on micro-optimizations -- making
every cycle, and every byte of generated code count. This forced us to take a
harder look at "string everywhere".

#### std::string Refactoring

Some common refactorings:

* Convert `static string const` to `char const[]`
  * Saves binary size, static ctors and dtors, and indirections to heap.
  * Generates temporaries when passed to `f(string const&)` which, remember,
    we have everywhere.
* Convert arguments from `string const&` to `char const*`.
  * Update virally...
  * Pay the strlen cost.
* Or, maintain two overloads. Not great.

I'm hoping that `[constexpr] string_view` will help here.

#### std::string Builders

Noob ex-Java programmers often wrote anti-patterns like:
```c++
  a = a + "_" + b;
```
Our profilers found these in droves.

So:
* We wrote concatenation helpers, and
* Fast type-safe printf like formatters.

The key was reserving capacity to minimize allocations.

These were "builders" a la Java's string builder. Keep in mind that the builder
pattern works well with *immutable* data. Immutability will come up again.

#### SSO and COW - locality

We compiled for both msvc and GCC. With msvc we had the SSO and we designed
around that. For example, we put short identifiers in strings. They lived on the
stack, or inline at the top of objects - a locality win. However, with GCC's
copy on write (COW) string we had reduced locality - a loss. In fact, this loss
drove us to a deep refactoring of these identifiers.

Lesson: locality is important.

#### SSO and COW - sharing

It turned out that our most memory constrained processes were built with GCC. In
one case when we faced a late-in-the-game ship stopping OOM condition the fix
was to carefully craft our code to "connect" the lifetime of strings and
preserve COW sharing.

OTOH reasoning about how COW sharing works can be non-intuitive:

TODO: code example, how reserve doesn't work as well as you'd like....

Lesson: sharing is important.

#### COW Bug

There's a subtle, but unresolved bug in GCC's COW implementation. We live with
it, but it haunts us every time we see memory corruption. So far, we have not
hit it. We think.

TODO: dig up GCC bug number.

The bug is exposed when getting *mutable* references to string. Necessarily
when the COW is saving us from the OOM we are sharing *immutable* strings --
otherwise there wouldn't be any sharing. So once again the idea of immutability
seems important, and related to sharing.

#### SSO vs. COW

They both have their place. We know that we're losing COW sooner rather than
later, but we also know that we have applications that depend on sharing. We have
some thinking and measuring to do...

Note that `fbstring` kept COW for large strings, though that was deprecated in
2016. On the other hand our ship stopper fix was for lots of *small* strings!

#### SSO and COW - size

SSO strings are **big**, 24 or 32 bytes. In our "`Optional<string>`" SSO was too
expensive, as most of our optional strings were unset. So, we specialized
`Optional<string>` to hold a `string *` -- saving space for the unset case but
adding an indirection / decreasing locality for the set case. That was
unfortunate.

On the other hand, a COW string was only one pointer, so with GCC we
conditionally compiled `Optional<string>` to not add the indirection.

Side note: A GCC COW string that's just `0` is not valid. We could implement
`Optional<string>` as a single pointer sized value. More on this later!

#### Intern strings

I mentioned above that COW string instances are smaller than SSO instances.
However, sometimes we wanted something even smaller than a pointer. For this
optimization, we built "tables" of de-duplicated strings. Clients use a small
integer "index" for each string.

We paid a cost to construct the table, but when were done we could build dense
"sets" of strings (indices) and manipulate them efficiently.

This is another example of immutability, an example of sharing, and an example
of a special allocator.

#### Heap fragmentation and large strings

TODO: Discuss how large contiguous strings have led to heap fragmentation.

#### Shrink vs. Reserve.

TODO: Discuss `std::string` reserve and *optional* (i.e. non-portable) shrink to
fit behavior.

#### Encodings

We've covered most of our wish list, but we also need to talk about encodings.

One class of serious bugs that we've face has been when an application
serializes invalid UTF-8 to the wire. Peer applications correctly reject such
data, leading to loss of communication.

Tracking an individual offending piece of code is usually straightforward. For
example we might find someone copying data directly from a device driver
without first checking the encoding. We can then fix that call site. We can
check encoding before serializing, but that's often distant from the offending
source of bad data. We have even done work to push checks further up the
"stack", but since we have "`std::string` everywhere" we are usually still
several steps away from the root cause when we detect this form of corruption.

Having a `utf8_string` would help us push checks back to the source of these
errors.

We also need the ability safely mutate UTF-8 along code point boundaries, and
reasonably handle illegal sequences.

### Formalizing

After reviewing bugs, our wish list, and other experience I built up a more
formal list of the "attributes" of strings. These next sections will review
these.

### Data

Any string implementation has to provide access to the data. This might be a
pointer to an "element" or a C-array of elements.

```c++
struct string_data {
  Elem* data;
};  
```

#### As a function
We can think of this as a free function:

```
string_get_data(s) -> s::element_type *;
```

### Length

String implementations are distinguished by how they store length:
* **In the type** - Like `array<int, 12>`.

```c++
   template<int Length>
   struct string_data {
      Elem data[Length];
   };
```
* **Next to the data** - Pascal strings anyone? Or GCC COW.
  strings?

```c++
   struct string_data {
      int length;
      Elem data[];
   };
```

* **Null termination (Embedded in the data)** - `const Elem*`.

```c++
   struct string_data {
      Elem data[];
   };
```

* **Next to a _reference_ to the data** - like `string_view`

```c++
   struct string_data {
      StringData *;
      int length;
   };
```

None of these strategies is "the best". They all have advantages and
disadvantages. For now, let us keep them *all* in mind.

#### Length Type

I used `int` above, but we have several choices for the *type* of the length:

* `size_t` - Do we really need the full address space?
* `int` - The "simplest" / "most obvious" alternative. Also, since we often do
  arithmetic with string length signed types are appropriate.
* `ptrdiff_t` - If we want "full size" but signed.
* `unsigned int` - If we want "reasonable size" but unsigned.

#### Empirical Length

TODO: Data on string lengths in memory and at rest from
some of our products. (maybe). Add war story about heap fragmentation?

#### As a function

So, given any string implementation there will be a function:

```
string_get_length(s) -> s::size_type;
```

### Continuity

TODO: Discuss continuity. I would like to limit discussion to contiguous
implementations. Strings *do* need to inter-operate with "ropes".

### Sharing

TODO: Add explanation of how sharing is typically implemented via reference
counting.

### Encoding

#### UTF-8 and Multi-byte Encodings

`std::string` completely ignores multi-byte encodings, but UTF-8 rules the real
world and real applications need to work in that world. `std::text_view` solves
part of the puzzle, but in practice the string type should know about encodings.

#### Illegal sequences

TODO: Discuss illegal sequences. ASCII and UTF both have them. Correction and
reporting.

#### ASCII

In many contexts, typically symbolic names, ASCII is an invariant.

ASCII is also a proper subset of UTF-8, and we can model that in the type
system:

```c++
struct utf8_string {
  // Implicit conversion:
  utf8_string(ascii_string const&);
  utf8_string& operator=(ascii_string const&);.
  // ... move, etc. ...
};
// or just slice:
struct utf8_string : ascii_string { ... };
```

#### UTF-16

TODO: We want UTF-16 too. Explain.

#### Legacy Encodings

TODO: Do more research and discuss Non-UTF, legacy encodings. Suggest that we
*not* consider these, but leave some room for compatibility via separate work?

TODO: More research on Unicode surrogates and UTF-32.

#### Code Points

TODO: Discuss `text_view` iteration, truncating at code point boundaries,
searching for code points, finding closest code point boundary, etc.

#### As a function

TODO: Need "function" form for encoding conversion.

```
string_get_encode(s) -> s::encoding_type;
encoding_is_multibyte(e) -> e::is_multibyte;
```

### Storage Duration
We have the four canonical storage durations:
* **Automatic** - Allocated by the compiler.
* **Static** - Allocated in program image by the compiler and linker.
* **Thread** - (Same as static for the purposes of this talk.)
* **Dynamic** - Allocated dynamically from a runtime library.

#### Automatic
Examples:
* C-array
* `string_view`.
* `folly::FixedString`
* `std::string` - sort of - when SSO is in play, or with heap elision?

#### Static

TODO: static. One example is refactoring const string to const char[].

#### Flavors of Dynamic

* **new/delete** - malloc/free.
* **Intern String** - special allocator.
* **Shared** - reference counted.

#### Non-type template parameters?

TODO: summarize current state of making strings as non-type template parameters.

### Null-ability

Most string implementations can represent null with "zero cost". nullptr for
`const char*`. `std::optional` could take advantage of a null string. Is there
an opportunity for a language extension for a null char[] literal?

### Mutability

TODO: More discussion on mutability, sharing and builders. Reducing the cost of
converting from immutable -> builder -> immutable.

### Summary Table

TODO: I will build a summary table or chart that compares the attributes
(enumerated above) of existing and potential string implementation types.

### Implementation

TODO: I will implement "toy" types that demonstrate possible compositions of
attributes and explore how they interact in various use cases.

I will provide examples of:

* Encoding as part of the type. Conversions to and from encodings including
  ASCII, UTF-8 and UTF-16.
* Handling of invalid sequences.
* Integration with `text_view`.
* Code point safe mutators.
* Compatibility and conversion between `constexpr string`, `string`, and `view`.
* Representing null.
* Shared immutable strings.
* Views into shared strings.
* Builders for immutable strings.
* Shared immutable strings in static storage.
* .. and more ...

### Conclusion

TODO:

### Q & A
