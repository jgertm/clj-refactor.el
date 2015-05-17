[![MELPA](http://melpa.org/packages/clj-refactor-badge.svg)](http://melpa.org/#/clj-refactor)
[![MELPA Stable](http://stable.melpa.org/packages/clj-refactor-badge.svg)](http://stable.melpa.org/#/clj-refactor)
[![Build Status](https://secure.travis-ci.org/clojure-emacs/clj-refactor.el.png)](http://travis-ci.org/clojure-emacs/clj-refactor.el)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/clojure-emacs/refactor-nrepl?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

# clj-refactor.el

`clj-refactor` provides refactoring support for clojure projects.

## Action shots

[add declaration](examples/add-declaration.gif)

[add import](examples/add-import.gif)

[add libspec](examples/add-libspec.gif)

[cycle coll](examples/cycle-coll.gif)

[cycle if](examples/cycle-if.gif)

[cycle privacy](examples/cycle-privacy.gif)

[cycle thread](examples/cycle-thread.gif)

[destructure keys](examples/destructure-keys.gif)

[expand let](examples/expand-let.gif)

[extract fn](examples/extract-fn.gif)

[create fn from example](examples/create-fn-from-example.gif)

[find usages](examples/find-usages.gif)

[introduce let](examples/introduce-let.gif)

[magic requires](examples/magic-requires.gif)

[move to let](examples/move-to-let.gif)

[promote fn 1](examples/promote-fn-1.gif)

[promote fn 2](examples/promote-fn-2.gif)

[rename symbol global](examples/rename-symbol-global.gif)

[rename symbol local](examples/rename-symbol-local.gif)

[thread first all](examples/thread-first-all.gif)

[thread fully unwind](examples/thread-fully-unwind.gif)

[thread last all](examples/thread-last-all.gif)

[thread last](examples/thread-last.gif)

[unwind thread](examples/unwind-thread.gif)

## Installation

I highly recommend installing clj-refactor through elpa.

It's available on [marmalade](http://marmalade-repo.org/) and
[melpa](http://melpa.org/):

    M-x package-install clj-refactor

## Setup

```el
(require 'clj-refactor)
(add-hook 'clojure-mode-hook (lambda ()
                               (clj-refactor-mode 1)
                               ;; insert keybinding setup here
                               ))
```

You'll also have to set up the keybindings in the lambda. Read on.

### Setup keybindings

All functions in clj-refactor have a two-letter mnemonic shortcut. For
instance, rename-file is `rf`. You get to choose how those are bound.
Here's how:

```el
(cljr-add-keybindings-with-prefix "C-c C-m")
;; eg. rename files with `C-c C-m rf`.
```

If you would rather have a modifier key, instead of a prefix, do:

```el
(cljr-add-keybindings-with-modifier "C-s-")
;; eg. rename files with `C-s-r C-s-f`.
```

If neither of these appeal to your sense of keyboard layout aesthetics, feel free
to pick and choose your own keybindings with a smattering of:

```el
(define-key clj-refactor-map (kbd "C-x C-r") 'cljr-rename-file)
```

**The keybindings suggested here might be conflicting with keybindings in
either clojure-mode or cider. Ideally, you should pick keybindings that don't
interfere with either.**

### Optional setup

#### Refactor nREPL middleware

For some of the more advanced refactorings we've written an [nrepl](https://github.com/clojure/tools.nrepl) middleware called
[refactor-nrepl](https://github.com/clojure-emacs/refactor-nrepl).

Add the following, either in your project's `project.clj`, or in the `:user`
profile found at `~/.lein/profiles.clj`:

```clojure
:plugins [[refactor-nrepl "1.0.5"]]
```

**WARNING** The analyzer needs to eval the code too in order to be able to build
  the AST we can work with. If that causes side effects like writing files,
  opening connections to servers, modifying databases, etc. performing certain
  refactoring functions on your code will do that, too.


#### Yasnippet
If you're not using yasnippet, then the "jumping back"-part of adding to
namespace won't work. To remedy that, enable the mode with either:

```el
(yas/global-mode 1)
```

or

```el
(add-hook 'clojure-mode-hook (lambda () (yas/minor-mode 1)))
```

It is an excellent package, so I recommend looking into it.

## Usage

This is it so far:

 - `ad`: add declaration for current top-level form
 - `ai`: add import to namespace declaration, then jump back
 - `ar`: add require to namespace declaration, then jump back (see optional setup)
 - `au`: add "use" (ie require refer all) to namespace declaration, then jump back
 - `cc`: cycle surrounding collection type
 - `ci`: cycle between `if` and `if-not`
 - `cp`: cycle privacy of `defn`s and `def`s
 - `dk`: destructure keys
 - `el`: expand let
 - `fe`: create function stub from example usage
 - `il`: introduce let
 - `mf`: move one or more forms to another namespace, `:refer` any functions
 - `ml`: move to let
 - `pc`: run project cleaner functions on the whole project
 - `pf`: promote function literal or fn, or fn to defn
 - `rf`: rename file, update ns-declaration, and then query-replace new ns in project.
 - `rl`: remove-let, inline all variables and remove the let form
 - `rr`: remove unused requires
 - `ru`: replace all `:use` in namespace with `:refer :all`
 - `sc`: show the project's changelog to learn about recent changes
 - `sn`: sort :use, :require and :import in the ns form
 - `sp`: Sort all dependency vectors in project.clj
 - `sr`: stop referring (removes `:refer []` from current require, fixing references)
 - `tf`: wrap in thread-first (->) and fully thread
 - `th`: thread another expression
 - `tl`: wrap in thread-last (->>) and fully thread
 - `ua`: fully unwind a threaded expression
 - `uw`: unwind a threaded expression

The following functions are available, and supported, but used rarely enough that they're not given a keybinding:

* `cljr-reify-to-record` change a call to reify with a call to a defrecord constructor.

[Using refactor-nrepl](#refactor-nrepl-middleware), you also get:

 - `am`: add a missing libspec
 - `as`: add implementation stubs for an interface or protocol
 - `ap`: add a dependency to your project
 - `cn`: Perform various cleanups on the ns form
 - `ef`: Extract function
 - `fu`: Find usages
 - `hd`: Hotload dependency
 - `rd`: Remove (debug) function invocations
 - `rs`: Rename symbol

Combine with your keybinding prefix/modifier.

### Customization

To take a look at the available settings do: `M-x customize-group <RET> cljr <RET>`

## Thread / unwind example

Given this:

```clj
(map square (filter even? [1 2 3 4 5]))
```

Start by wrapping it in a threading macro:

```clj
(->> (map square (filter even? [1 2 3 4 5])))
```

And start threading away, using `cljr-thread`:

```clj
(->> (filter even? [1 2 3 4 5])
     (map square))
```

And again:

```clj
(->> [1 2 3 4 5]
     (filter even?)
     (map square))
```

You can also do all of these steps in one go.

Start again with:

```clj
(map square (filter even? [1 2 3 4 5]))
```

Put your cursor in front of the s-exp, and call `cljr-thread-last-all`:

```clj
(->> [1 2 3 4 5]
     (filter even?)
     (map square))
```

There is a corresponding `cljr-thread-first-all` as well.

To revert this, there's `cljr-unwind` to unwind one step at a time. Or
there's `cljr-unwind-all` to unwind the entire expression at once.

To see how that works, just read the examples in the other direction.

## Introduce / expand / move to let example

Given this:

```clj
(defn handle-request
  {:status 200
   :body (find-body abc)})
```

With the cursor in front of `(find-body abc)`, I do `cljr-introduce-let`:

```clj
(defn handle-request
  {:status 200
   :body (let [X (find-body abc)]
           X)})
```

Now I have two cursors where the `X`es are. Just type out the name,
and press enter. Of course, that's not where I wanted the let
statement. So I do `cljr-expand-let`:

```clj
(defn handle-request
  (let [body (find-body abc)]
    {:status 200
     :body body}))
```

Yay.

Next with the cursor in front of `200`, I do `cljr-move-to-let`:

```clj
(defn handle-request
  (let [body (find-body abc)
        X 200]
    {:status X
     :body body}))
```

Again I have two cursors where the `X`es are, so I type out the name,
and press enter:

```clj
(defn handle-request
  (let [body (find-body abc)
        status 200]
    {:status status
     :body body}))
```

Pretty handy. And it works with `if-let` and `when-let` too.

## Destructuring keys

Given this:

```clj
(defn- render-recommendation [rec]
  (list [:h3 (:title rec)]
        [:p (:by rec)]
        [:p (:blurb rec) " "
         (render-link (:link rec))]))
```

I place the cursor on `rec` inside `[rec]` and do `cljr-destructure-keys`:

```clj
(defn- render-recommendation [{:keys [title by blurb link]}]
  (list [:h3 title]
        [:p by]
        [:p blurb " "
         (render-link link)]))
```

If `rec` had still been in use, it would have added an `:as` clause.

For now this feature is limited to top-level symbols in a let form. PR welcome.

## Stop referring

Given this:

```clj
(ns cljr.core
  (:require [my.lib :as lib :refer [a b]]))

(+ (a 1) (b 2))
```

I place cursor on `my.lib` and do `cljr-stop-referring`:

```clj
(ns cljr.core
  (:require [my.lib :as lib]))

(+ (lib/a 1) (lib/b 2))
```

## Promote function

Given this:

```clj
(map #(-> % (str "!") symbol) '[aww yeah])
```
And I place my cursor on `symbol` and do `cljr-promote-function` and rename `%` to `sym`:

```clj
 (map (fn [sym] (-> sym (str "!") symbol)) '[aww yeah])
```

And I place my cursor on `symbol` and do `cljr-promote-function` and call the fn `shout-it!`

```clj
(defn shout-it!
  [sym]
  (-> sym (str "!") symbol))

(map shout-it! '[aww yeah])
```

With a prefix it will promote a function literal all the way to a defn.

## Magic requires

Common namespace shorthands are automatically required when you type
them:

For instance, typing `(io/)` adds `[clojure.java.io :as io]` to the requires.

- `io` is `clojure.java.io`
- `set` is `clojure.set`
- `str` is `clojure.string`
- `walk` is `clojure.walk`
- `zip` is `clojure.zip`

You can turn this off with:

```el
(setq cljr-magic-requires nil)
```

or set it to `:prompt` if you want to confirm before it inserts.

## Create function stub from example usage

When you have a function in mind that you eventually want to create you can just type out its signiture as it is used, for example:

```clojure
(defn my-lovely-fn [some-param]
  (let [foo (util-fn some-param)]
    (other-fn some-param foo)))
```

Then invoking create function stub from example `fe` will create the appropriate stub for you:

```clojure
(defn- other-fn [some-param foo]
  )

(defn my-lovely-fn [some-param]
  (let [foo (util-fn some-param)]
    (other-fn some-param foo)))
```

In case your params for the new function are not symbols the feature will replace them with a generated param name:

```clojure
(defn- other-fn [some-param arg1]
  )

(defn my-simple-fn [some-param]
  (other-fn some-param (util-fn some-param)))
```

(Use rename symbol on the argument afterwards if you wish.)

## Project clean up

`cljr-project-clean` runs some clean up functions on all clj files in a project
in bulk. By default these are `cljr-remove-unused-requires` and `cljr-sort-ns`.
Additionally, `cljr-sort-project-dependencies` is called to put the
`project.clj` file in order. Before any changes are made, the user is prompted
for confirmation because this function can touch a large number of files.

This promting can be switched off by setting `cljr-project-clean-prompt` nil:

```el
(setq cljr-project-clean-prompt nil)
```

The list of functions to run with `cljr-project-clean` is also configurable via
`cljr-project-clean-functions`. You can add more functions defined in
clj-refactor or remove some or even write your own.

`cljr-project-clean` will only work with leiningen managed projects with a
project.clj in their root directory. This limitation will very likely be fixed
when [#27](https://github.com/magnars/clj-refactor.el/issues/27) is done.

## Add project dependency

Easily add new project dependencies. Completing read among artifacts
from clojars and a selection from maven central, following by a
completing read of versions and insertion into `project.clj` and
hotloading into the repl.

When this function is called with a prefix the artifact cache is invalidated and
updated. This happens synchronously. If you want to update the artifact cache in
the background you can call `cljr-update-artifact-cache`.

The variable `cljr--hotload-dependencies` defaults to `true` and
determines if new dependencies should be hotloaded or not.

## Add missing libspec

`cljr-add-missing-libspec` will try to resolve the symbol at point and require or import the missing var.  If there are more than one var match the user is prompted. If there's only one result it is added without user intervention.  If the symbol at point is of the form `edn/read-string` the resulting libspec will be aliased to `edn`.

## Remove (debug) function invocations

Removes invocations of a predefined set of functions from the namespace. Remove
function invocations are configurable `cljr-debug-functions`; default value is
`"println,pr,prn"`.

## Find usages

Opens a grep buffer, clickable listing all occurrences of the symbol (most likely def or defn). The project's namespaces don't need to be loaded although cider needs to be able to resolve the symbol you want to search for.

## Rename symbol

Renames the symbol at the cursor across the project. **WARNING** Also reloads the buffer as a side effect where the symbol is defined *only* if that namespace was already loaded. This is needed so the ASTs for the changed files can be recreated (they will refer the renamed symbol).

## Clean ns

This op performs the following cleanups of the ns form:

* Eliminate :use clauses
* Sort required libraries, imports and vectors of referred symbols
* Rewrite to favor prefix form, e.g. [clojure [string test]] instead
  of two separate libspecs
* Raise errors if any inconsistencies are found (e.g. a libspec with more than
  one alias.
* Remove any duplication in the :require and :import form.
* Remove any unused libspec vectors
* Remove unused symbols from the :refer vector, or remove it completely.
* Remove any unused imports

Note that we have to build an AST in order to do this, so if the current file is in a bad state this op won't work.

## Hotload dependency

If point is on a dependency vectors in, e.g. in `project.clj`, it will get hotloaded into the repl.


## Extract function

Extract the form at point, or the nearest enclosing form, into a toplevel defn.

With a prefix the newly created defn will be public.

## Changing the way the ns declaration is sorted

By default sort ns `sn` will sort your ns declaration alphabetically. You can
change this by setting `cljr-sort-comparator` in your clj-refactor
configuration.

Sort it longer first:

```el
(setq cljr-sort-comparator 'cljr--string-length-comparator)
```

Or you can use the semantic comparator:

```el
(setq cljr-sort-comparator 'cljr--semantic-comparator)
```

The semantic comparator sorts used and required namespaces closer to the
namespace of the current buffer before the rest. When this is not applicable it
falls back to alphabetical sorting.

For example the following namespace:

```clj
(ns foo.bar.baz.goo
  (:require [clj-time.bla :as bla]
            [foo.bar.baz.bam :refer :all]
            [foo.bar.async :refer :all]
            [foo [bar.goo :refer :all] [baz :refer :all]]
            [async.funkage.core :as afc]
            [clj-time.core :as clj-time]
            [foo.async :refer :all])
  (:import (java.security MessageDigest)
           java.util.Calendar
           [org.joda.time DateTime]
           (java.nio.charset Charset)))
```

will be sorted like this:

```clj
(ns foo.bar.baz.goo
  (:require [foo.bar.baz.bam :refer :all]
            [foo.bar.async :refer :all]
            [foo.async :refer :all]
            [foo [bar.goo :refer :all] [baz :refer :all]]
            [async.funkage.core :as afc]
            [clj-time.bla :as bla]
            [clj-time.core :as clj-time])
  (:import (java.nio.charset Charset)
           (java.security MessageDigest)
           java.util.Calendar
           [org.joda.time DateTime]))
```

The `cljr-sort-comparator` variable also enables you to write your own
comparator function if you prefer. Comparator is called with two elements of the
sub section of the ns declaration, and should return non-nil if the first
element should sort before the second.

## Automatic insertion of namespace declaration

When you open a blank `.clj`-file, clj-refactor inserts the namespace
declaration for you.

It will also add the relevant `:use` clauses in test files, normally using
`clojure.test`, but if you're depending on *midje* or *expectations* in your
`project.clj` it uses that instead.

Like clojure-mode, clj-refactor presumes that you are postfixing your
test files with `_test`.

Prefer to insert your own ns-declarations? Then:

```el
(setq clj-add-ns-to-blank-clj-files nil)
```

## Miscellaneous

### Tweaks to paredit

With clj-refactor enabled, any keybindings for `paredit-raise-sexp` is
replaced by `cljr-raise-sexp` which does the same thing - except it
also removes any `#` in front of function literals and sets.

### Reload config

You can use `cljr-reload-config` to resubmit configuration settings
to `refactor-nrepl` after changing them in emacs.  This is a good
alternative to restarting the repl whenever you change settings
affecting the middleware.

Currently the only setting that applies to the middleware is
`cljr-favor-prefix-notation`.

## More stuff to check out

You might also like

- [align-cljlet](https://github.com/gstamp/align-cljlet) - which is an Emacs package for aligning let-like forms.
- David Nolen has created some [clojure-snippets](https://github.com/swannodette/clojure-snippets)
- Magnar Sveen made some [datomic-snippets](https://github.com/magnars/datomic-snippets)
- Max Penet has also created some
   [clojure-snippets](https://github.com/mpenet/clojure-snippets), early fork of
   dnolens' with tons of additions and MELPA compatible

## Changelog

An extensive changelog is available [here](CHANGELOG.md).

## Contribute

Yes, please do. There's a suite of tests, so remember to add tests for your
specific feature, or I might break it later.

You'll find the repo at:

    https://github.com/clojure-emacs/clj-refactor.el

To fetch the test dependencies, install
[cask](https://github.com/cask/cask) if you haven't already,
then:

    $ cd /path/to/clj-refactor
    $ cask

Run the tests with:

    $ ./run-tests.sh

## Contributors

The project is maintained by

- [Magnar Sveen](https://github.com/magnars)
- [Lars Andersen](https://github.com/expez)
- [Benedek Fazekas](https://github.com/benedekfazekas)
- [AlexBaranosky](https://github.com/AlexBaranosky)

Additional contributors are:

- [Bozhidar Batsov](https://github.com/bbatsov)
- [Ryan Smith](https://github.com/tanzoniteblack)
- [Hugo Duncan](https://github.com/hugoduncan)
- [Luke Snape](https://github.com/lsnape)
- [Aaron France](https://github.com/AeroNotix)

Thanks!

## License

Copyright © 2012-2015 Magnar Sveen

Authors: Magnar Sveen <magnars@gmail.com>
Keywords: clojure convenience

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
