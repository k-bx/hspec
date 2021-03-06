---
layout: default
---

## Automatic spec discovery

It is a useful convention to have one spec file for each source file.  That way
it is straightforward to find the corresponding spec for a given piece of code.
So let's assume we have a project with the following directory layout:

```
src/
├── Foo.hs
├── Foo/
│   └── Bar.hs
└── Baz.hs
test/
├── FooSpec.hs
├── Foo/
│   └── BarSpec.hs
└── BazSpec.hs
```

The `src` directory contains three module `Foo`, `Foo.Bar` and `Baz`.  The
`test` directory contains corresponding specs `FooSpec`, `Foo.BarSpec` and
`BazSpec`.

Now if we want to run all specs in one go we have to write the following
boilerplate:

```hspec
import Test.Hspec

import qualified FooSpec
import qualified Foo.BarSpec
import qualified BazSpec

main :: IO ()
main = hspec $ do
  describe "Foo"     FooSpec.spec
  describe "Foo.Bar" Foo.BarSpec.spec
  describe "Baz"     BazSpec.spec
```

This is error prone, and neither challenging nor interesting.  So it should be
automated.  Hspec provides a solution for that.  We make creative use GHC's
support for custom preprocessors.  The developer only has to creates a
_test driver_ that contains a single line:

```haskell
-- file test/Spec.hs
{-# OPTIONS_GHC -F -pgmF hspec-discover #-}
```

This instructs GHC to invoke `hspec-discover` as a preprocessor on
the source file.  The rest of the source file is empty, so there is nothing to
preprocess.  Rather than preprocessing, `hspec-discover` scans the file system for all spec
files belonging to a project and generates the required boilerplate.
`hspec-discover` does not parse any source files, it instead relies on the
following conventions:

 * Spec files have to be placed into the same directory as the test driver, or
   into a subdirectory.
 * The name of a spec file has to end in `Spec.hs`; the module name has to
   match the file name.
 * Each spec file has to export a top-level binding `spec` of type {{'Spec'|id}}.

A complete example is at <https://github.com/sol/hspec-example>.
