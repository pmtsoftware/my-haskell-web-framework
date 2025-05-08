# my-haskell-web-framework

Series of blog posts about creating web framework in Haskell from scratch.

## Introduction

We're going to build and test our own web framework. Later we will create some simple app
and check how we can scale it and what are the limits.

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
Let's call it **swf** as abbreviation of Simple Web Framework:
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
Let's add `NoImplicitPrelude` extension to `package.yaml`:
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
So far so good!

## HTTP server

I'm going to use [scotty](https://github.com/scotty-web/scotty).
Add scotty to dependencies list in executables section of `package.yaml`:
```
executables:
  swf-exe:
    main:                Main.hs
    source-dirs:         app
    ghc-options:
    - -threaded
    - -rtsopts
    - -with-rtsopts=-N
    dependencies:
    - swf
    - scotty
```
Run `stack build`.
Now let's turn our tui _hello-world_ app into web app. Run `scotty` with proper GET handler in `Main.hs`:
```
{-# LANGUAGE OverloadedStrings #-}

module Main (main) where

import Relude

import Lib

import qualified Web.Scotty as Scotty

main :: IO ()
main = Scotty.scotty 3000 $
    Scotty.get "/" $ do
        Scotty.text "Hello World!!!"
```

Because non-root user cannot bind to port 80 we use port 3000 instead. So open browser and type _127.0.0.1:3000_ in address bar to see 
"Hello world!!!" message. Let's move `OverloadedStrings` extension to `package.yaml`. We will use it very often so it's wise to enable it for all modules.
Also `import Lib` is unnecessary so removing it will get rid of compiler warning.

[PR](https://github.com/pmtsoftware/swf/pull/1)

## Rendering HTML

Web application which renders only text is quite poor so let's introduce some HTML. We will use blaze-html which is a [combinatory library](https://en.wikipedia.org/wiki/Combinator_library). Most web frameworks use some kind of templating system (like jsx, etc.) but in our framework building html is just as simple as writing 
haskell code. Let's also add bootstrap styles to make our html a little bit nicer.

Commit: [ffba758](https://github.com/pmtsoftware/swf/commit/ffba75824a1b3ec9dd330a8e65dc2036562905a4).

## Database

We will use PostreSQL as our database. For now all our framework needs is connection string to postgresql database.
For development purpose I'm going to use local server. To install just follow installation instruction.
In order to create database use `psql` tool. In terminal type `sudo su postgres` and then `psql`.

Create database:
```
CREATE DATABSE swf;
```
Create database user:
```
CREATE USER swf PASSWORD 'swf';
```
Add access to database:
```
GRANT ALL PRIVILEGES ON DATABASE swf TO swf;
```
In order to connect `swf` user to db server it might be necessary to modify `/var/lib/pgsql/data/pg_hba.conf` file.
In my case I changed method to `trust` for local Unix domain socket connections:
```
local   all             all                                     trust
```
Restart postgresql service:
```
sudo systemctl restart postgresql
```
Connect to db `psql -d swf -U swf`.
Database is ready.
