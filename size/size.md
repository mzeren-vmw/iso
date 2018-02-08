# Bio

Mark is a staff engineer at VMware where he works on C++ libraries, coding
standards, and toolchains.

* First Name: Mark
* Last Name: Zeren
* Email: mzeren@vmware.com
* Twitter Handle: @markzeren
* Organization: VMware Inc.
* Country: US

# Session Level

* [x] Beginner
* [x] Intermediate
* [ ] Advanced
* [ ] Expert

Session Type: Case study

Preferred Session Length: 45

Minimum Session Length: 45

Maximum Session Length: 90

# Tags

* Please include a comma separated list of at least three tags for the session.
Use lower case except for proper nouns and acronyms. Strive for the general, not
the specific:"testing," rather than "unit tests." Spell out acronyms unless they
are more likely to be understood as acronyms: "artificial intelligence" rather
than "AI" and "GUI" rather than "graphical user interface." For example,
"games,performance,cross platform,I/O,C compatibility,HTML" For additional
information, see our tagging guide.

code reviews, code metrics, legacy code

# Audience Description

* Examples: library authors, application developers, game developers, etc. A
quality description helps your audience find your submission. Please keep it
short and sweet. Full sentence not required.

* Developers interested in code review process and code health.
* C++ language experts interested in seeing the effects of various C++ constructs.

# Title

-Os Matters

# Session Description

At VMware we include binary size deltas in code reviews for large, C++,
user-space, applications. You might be thinking "that's the most pointy haired
thing I've ever heard of!". Come to this talk and learn how this
metric provides surprisingly strong counter pressure to complexity.

# Outline

## Acknowledgement to Ole Agesen

Ole is a Fellow at VMware, and this is his brain child.
Any mistakes or misconceptions are mine.

## The Graph

(The talk will progressively reveal and annotate a graph that shows binary size
over time.)

## Fast Growth Company

(The first reveal of the graph is the far lefthand side that shows a steady
increase in binary size over time.)

* We add features, ergo
* We add code.
* We under-allocate tech debt cleanup, ergo
* We never remove code.
* (Also we use C++ so it's big!)

## But we don't even see the graph

.. or the size. What we see:
* Code is brittle,
* Code is slow.
* Code is hard to navigate and understand.
* Contributors add their feature and run.
* Etc.

Everyone in the room has seen this.

## Motivated to Fix

We are motivated to fix, but we have:
* Many lines of attack.
* Many opinions.

Requires:
* Staffing
* Leadership
* Workflow

## Herding cats

One of the hardest problems in software engineering: The Herding Problem.

How do we collectively make forward progress?

## Metrics

Herd with metrics.

Of course we measure things you'd expect:
* Ops/second
* Resources / Operations
* Max capacity
* Uptime
* Micro-benchmarks
* Coverage

But these don't really drive reduction in _complexity_
(Except maybe coverage)

Some of them can even drive increases in complexity.

Many of these were only available to us as integration tests, which were too
heavyweight for high velocity code edits.

## Binary Size

Enter binary size.

## Binary Size in Practice

(Example of adding to code review)

(Discussion of calculation)

## Early Wins

Describe a couple of early wins. One example is that we were accidentally
compiling with Boost asserts in release builds. No-one noticed.

## Instability

Discuss how `-O2` is too unstable for a single change. Mostly due to variance in
inlining. Includes pretty graph.

The solution was to switch to `-Os`.

## Expansion of the graph

The following slides reveal more and more of the graph describing macro
steps and stages. For example:

* Removing test and instrumentation code.
* Garbage collecting dead code at link time.
* Symbol visibility.
* Optimizing emitted code.
* Optimizing widely used library code.
* Upgrading compilers.
* Upgrading to `-std=c++11`
* Size increase for 32bit -> 64bit conversions.
* Etc.

-> These generate discontinuities. They are all good to do, but we also need to
change the behavior at a lower level.

(Also talk about size increases during "freeze for release".)

## Individual Changes and Anecdotes

The following slides discuss how measuring size affects developing and reviewing
individual changes. There are many examples to show, I'm not sure which
ones will make it into the final presentation.

* Simplifying class hierarchy
* Simplifying templates.
* bind -> lambda
* Range based for conversions.
* Working with undefined behavior.
* Exception throwing.
* Instrumentation (profile counters) in release builds.
* `default` in switch statements (and -Wswitch).
* Boost Asio symbol visibility.
* GCC dtor/ctor de-dup.
* Module static functions.
* Object copies.
* shared_ptr.
* unique_ptr.
* std::list::size (in older gcc).
* stringstream.
* Etc.

-> These little steps, and avoiding missteps, are the real, and ultimately
transformative, effect.

## Counter indications

The rule is to measure, but the measurement is not the rule.

This section discusses examples where binary size needs to be considered more
thoughtfully:

* range based for loops
* bind -> lambda
* emitted code for public APIs
* static constructors.
* RAII.
* Algorithms.
* unique_ptr.
* Etc.

## Mocking and Unit Tests

* Mocking in C++ is not easy.
* Mocking with 0 size impact is harder.

## Shipping -Os

Some binaries shipping with -Os. Discuss:
* Performance
* Workflow/build simplification.

## Tools

Describe scripts and tooling that we use.

Maybe I will open source one of these.

Maybe I will write new open source scripts around
https://github.com/google/bloaty.

# Identical Code Folding

Show how ICF affects the binary size metric.

## LTO

LTO has a big effect on binary size. Show numbers. Discuss instability.

## Community Context

* godbolt.org and "knowing your disassembly".
* Jason Turner's Commodore 64 CppCon '16 keynote.
* Bloaty (if not discussed above).

## Where are we now?

Final part of the graph is going up? What's going on here.
* We are adding features!
* And I'd like to think faster and at lower cost.
* But it's also cyclical.

## Conclusion and Future Work

Some discussion of future work. For example, how this metric interacts
with new language features, particularly `constexpr`.

## Q&A
