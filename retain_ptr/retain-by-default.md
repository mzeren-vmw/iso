# Retain by default

Passing a pointer generates better code than passing a smart pointer by
reference, so we want to pass by pointer. Once one reaches that conclusion,
retain by default is the most natural way to use retain_ptr.

Here's an example. For brevity I'm leaving out `const`, using only `struct` and
compiling with -Os. The argument holds for -O3 as well. You can edit the code on
[Godbolt.org](https://godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAM1QDsCBlZAQwBtMQAGAOgFZOBgoUICMpAFYgxqAA4E8dAM6U0rVAFdieRZgCCigLYgA5J2Ok0BmXnYB5WgGEEzWsA6njASlKKNxZO4ApABMAMx4tMis6lgA1IGhDsiKBPioCdiBnLoh4ZHRcQkOKVquGVk52eqKEcCxtMwGmIoyzAGxKegJAEIVFSXqyASxACIA7szEVPEA7L3ZsbEAbnjEBOpsscjOxFtKBABUsW4EAHKNmBCes92xxJjrxLTxwcEAYjZ4IcE9syN9M3%2BoXmlX0BGIg2G9wIzAiAH1mAQYds4cNAnN0UCQRUCJgrKxEZgilFmIpFLEACrlbIDIZ3B6w2hwuS7dEgxbQxnM8FXWIgWLc4gQWjqVisFnXNmYiochnwlkQCmxA4SvkChWqtmxAD0R05EQggs8vwO2r%2BMvpMPl4JCADZYrJMMRESQEiNFUdNRiFotLVyWbECFYIBKehbFtVah1UiAQIoJjIwz7FvHmDIIEGZKRlQQENpjcDw5bNM8DrntEndItpT6AH76pkKyVzHV6zDsUmXI0ms01qt%2B61ChsIpFtBCo7NKlXg678wUhmc3PuLJaoPDoWKI5EID2xL3s326ge0BfEAu3U1Fo/3Ds6Q2h4HK7VFwXxUIjPc2ws%2B5eUo6O50iGIABaDJeS1aES3VL9bl/CkjhOcCW0gp5oOIX44KOQVK2lb9QVyPAqCwaZul0AAlbA4QABVsABJU4KWwMiKlXddjnuB4IHGSYqCOdAJimY1slydsdBYtcN2ADiCAgYcWSKbipnKYJ7X4nihJyMJMFofAqABA8ZFKAgqAgb4AAl2zUF5eEUb5szUqZQNCbATnOJornPAF/mEmlwUhWJsAADxkTB8HkOglx/MJCOI2JSIo6i6IYpiLUU3jjgeJgNBk5tbhQ544UUNB1AIbhEPPc0otCUSiR9OSvwcNLlPtE4spKpC8oeKDCuKggMMBC0RJ0wiLVYjcdEYXquIE9Kiuy3KBTmkq3w/Ja%2BsfX96vQxImucxbepwga8PEtiUkmHL9ItIKQrChRnnEPxaEwABPbBSXWg8CKIzASPIyiaPoxjmJ9KTME4h6Sxet6UjKzKps8Tyqpqi1QfBx6ofe2HJuyjzK2rLThr04SBp8sFETwZBj2wnaZoyDoHGYJYiTwsbYkwYK1HuBmmYga7QsIO67TZjSpR9RRucwFa2axtqLrwxZMG4CbZeFUVxRnQ7vNBdncVQ1nFJWWgAFkImaOEHHuQlpp4g4DkR0EtoU2m9vuZAIUIK59PiH00r452DwNiITaexRzct3FTJUhyqHt2VHmeNlhy3MdUSuezaaO7ESd0LxSFYExeHMWgTBEcxUBMBwQl6YJbl8TR2lyUvSAIEwzAR0gAGsQFtERuAADk4XgABYVNCW0%2B4ATlCYIh4nvOTCHovW7LkxzGUThm%2BX7w4FgJBLGsdgyAoCB95sJ0QGAGZFAaGRFAQVACFIKgbF15QIAAI2XixUAMJp6HsVgz0v74FdvIJmygPDeBoPQJgbAOA8H4MIJBAhpByDuhA4CthyTAU6G6ZAVcq4iAAF6xGAiMeK/0kpA1zvnYwhdSDFw8KQcuxhG6xDGIQBAsRAp91tMBW0Q9jjIEpjMWIEBr5pjvg/Tw5gW6QO8F3GY3AZ4CAEWEEQMxJ5hFtPPYwi8GFfxYWvLgm9IGkB3ogFAP8D5OnICoaxZ9iAXyvjfKRj9n6sFfpQT%2BTDLB/wIAAoBTCQGYCGHgcBW8n50EmnArgfBkHINQeFWgGCsGkNwe%2BfBNdCEkJoQXJeTCWFsI4bmbhvD%2BGCOAMI2IojxGuPvgQGRpi24KJAKEHgE9h78CnhPCewRNGhF0foxhZhmGrx8CYuRLTdHBAKaMoxzSvDeCZsQGodAQBDyAA%3D)

Start with some boilerplate:

```C++
#include <cstdio>
#include <string>

using namespace std;

struct Dwarf {
  virtual char const* Name() { return "Fili"; }
};

template<class T>
struct retain_ptr {
  T* operator->() { return _ptr; }
  T* get() { return _ptr; }
  T* _ptr;
};
```

Now look at a function that takes a pointer:

```C++
void greet(Dwarf* dwarf)
{
  printf("Hello %s", dwarf->Name());
}
```

It will generate the following code:

```
 1 greet(Dwarf*):
 2         subq    $8, %rsp
 3         movq    (%rdi), %rax
 4         call    *(%rax)
 5         popq    %rdx
 6         movq    %rax, %rsi
 7         movl    $.LC0, %edi
 8         xorl    %eax, %eax
 9         jmp     printf
```

Now look at a function that takes a `retain_ptr&`:

```C++
void f(retain_ptr<Dwarf>& dwarf)
{
  printf("Hello %s", dwarf->Name());
}
```

It will generate:

```
 1 greet(retain_ptr<Dwarf>&):
 2         subq    $8, %rsp
 3         movq    (%rdi), %rdi
 4         movq    (%rdi), %rax
 5         call    *(%rax)
 6         popq    %rdx
 7         movq    %rax, %rsi
 8         movl    $.LC0, %edi
 9         xorl    %eax, %eax
10         jmp     printf
```

Look at line 3. We have an extra indirection.

This might seem like a small thing, but it will add up in a large codebase.

#### Extrapolating from there

If we should always pass by pointer, then we should return by pointer too:

```c++
struct Expedition {
  Dwarf* getScout() { return _scout.get(); }
  retain_ptr<Dwarf> _scout;
};

void start()
{
  Expedition journeyEast;
  greet(journeyEast.getScout());
}
```

Of course the `Dwarf*`'s lifetime is scoped to the Expedition's lifetime.

#### Retaining arguments

When we want to retain a result or a passed in argument `operator=`
is the natural tool to use:

```c++
struct Expedition {
  ...
  void setScout(Dwarf *scout) { _scout = scout; }
  ...
};

static retain_ptr<Dwarf> sCave;
void exploreCave(Expedition& e)
{
  sCave = e.getScout();
  e.setScout(nullptr);
}
```

Which leads to the following pattern:

```c++
template<class T>
struct retain_ptr {
  retain_ptr() : _ptr(nullptr) {}
  retain_ptr(T* ptr) : _ptr(ptr) { /* retain(_ptr); */ }
  retain_ptr& operator=(T* ptr) {
    retain_ptr tmp(ptr);
    using std::swap;
    swap(tmp, *this);
    return *this;
  }
  ~retain_ptr() { /* release(_ptr); */ }
  T* operator->() { return _ptr; }
  T* get() { return _ptr; }
  T* _ptr;
};
```

#### Attach / Detach still required

Of course we still need an "attaching" constructor and mutator, because often an API will give us pre-retained "poitners". Something like:

```c++
struct retain_attach_t {};

template<class T>
struct retain_ptr {
  ...
  enum attach_t { attach_only };
  retain_ptr(retain_attach_t, T* ptr) : _ptr(ptr) {}
  void attach(retain_attach_t, T* ptr) {
    /* retain(ptr); */
    /* release(_ptr); */
    _ptr = ptr;
  }
  ...
};

extern void Adventure_CreateDwarf(Dwarf**);

retain_ptr<Dwarf> recruit()
{
  Dwarf* dwarf;
  Adventure_CreateDwarf(&dwarf);
  return {retain_attach_t(), dwarf};
}
```

But these will only be used at the interface with the underlying "native" APIs,
far less often than normal parameter passing and result returning.
