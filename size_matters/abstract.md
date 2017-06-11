# -Os matters, but not how you think

Measuring binary size at every commit has changed the way I think about C++, and
it helped my team shrink a large application in half. Binary size, measured with
-Os, serves as a complexity metric -- not a perfect one, but one that it is
readily at hand and easily understood. Adopting the dicipline of reporting size
changes with every code review is not necessarily easy, it may even sound
hairbrained, but before you dismiss it, come to this talk to see its
effects on a whole project as well as on individual lines of code.

# Outline

# Results
This talk is backwards. It starts with the results.
* Set the scene with original binary size.
* Number of API types and functions.
* Show graph of binary size reduction over time.
  * Details on all the bumps to come later...

# Size diffs in reviews
* Review culture.
* Every change includes size data.

# Workflow
* Two work trees.
* Wrapper for Linux `size`

# Working locally
* Measuring locally changes what you write, even before review.

# More discussion of reviews
* Increases are OK, but need some justification

# Bumps
The following sections explain varioius changes and phases in the graph.

## -DNDEBUG
* We don't care about debugging code.

## Compilers
* Gcc 4.8
* -std=c++11

## -0s
* `-Os` is more stable than `-O2` or `-O3`

## Removing old code
* Test harnesses that ship in production
* ...

## GC sections
* ...

## Code generators
* ...

## Class hierarchy
* ...

## std::bind, std::function
* lambda ...

## Boost
* ...

## Exception throw helpers
* ...

## iostream
* ...

## Logging
* ...

## Unused paramters
* ...

## Return value vs. out parameter
* ...

## range for loop
* ...

## Etc.

# Advaned tools
* scripts for diffing disassembly

# Notes about Windows
* ...
