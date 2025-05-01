# my-haskell-web-framework

Series of blog posts about creating web framework in Haskell from scratch.

## Introduction

We're going to build and test our own web framework. Later we will create some simple app
and check how we can scale it and what is the limit.

## Prerequisites

Dev environment:

1. OS: I use [Fedora](https://fedoraproject.org/) but you can use any linux distribution.
In fact it should work also on BSD-like systems, MacOS and Windows (if you like video games :))

2. Haskell platform: when on linux I use [ghcup](https://www.haskell.org/ghcup) to install haskell tools.
After installation run `ghcup tui` to install [GHC](https://www.haskell.org/ghc/), [cabal](https://www.haskell.org/cabal/), [Stack](https://docs.haskellstack.org/) and [HLS](https://github.com/haskell/haskell-language-server).
I use following versions:
* GHC: 9.6.6
* cabal: 3.12.1.0
* Stack: 3.3.1
* HLS: 2.9.0.1
Try `ghc --version` in terminal. If everything works you should see something similar to `9.4.8`.

3. IDE: Personally I prefer vim or nvim but you can go with something more modern like VS Code or cursor.
Generally any text editor is sufficient.

That's the bare minimal stuff we need to start. Other tools (like database etc.) we will install when we need them.

## Hello world

Let's start with something really simple. In first step we need to setup our project.
Let's call it **swf** as abbreviation of Simple Web framework:
```
stack --resolver lts-21.25 new swf new-template`.
cd swf
stack build
```
Next modify `Lib.hs` :
```
module Lib
    ( someFunc
    ) where

someFunc :: IO ()
someFunc = putStrLn "Hello world!"
```

Run it:
`stack build && stack exec swf-exe`

Good job!

Code is also available Github [repository](https://github.com/pmtsoftware/swf).

## Get rid of Prelude

Because standard library called `Prelude` is full of stuff we won't use we gonna replace it with something more useful.
I prefer `Relude` but there are many good alternatives. With `Relude` we're not only avoiding polluting our namespace with unneeded functions but also
reimporting things from packages like `containers`, `text`, etc. what means less `import`s in our codebase.
Let's add `NoImplicitPrelude` extension to `package.yaml` file:
```
default-extensions:
- NoImplicitPrelude
```
and `relude` package to dependencies list:
```
dependencies:
- base >= 4.7 && < 5
- relude
```
But now our code breaks due to fact that GHC doesn't know where to find `putStrLn` function.
We have to add `import Relude` to `Lib.hs` and `Main.hs`.
Now `stack build` finishes successfully.
Looks good!
