NOTE: THIS REPO IS PRIVATE BUT HAS PUBLIC CONTRIBUTORS!
=======================================================

-----

Reason: Meta Language Utility
=========================================

- Approachable ES6 style syntax.
- Powerful, automatic source code formatting.
- Adopt Incrementally with straightforward `JavaScript/C` interop.
- Bare metal compilation - *No Virtual Machine*.
- Type inference, pattern matching, immutable by default.

Install Stable
----------

```sh
brew update
brew install opam --HEAD
opam init
# Add this to your ~/.bashrc (or ~/.zshrc):
#   eval `opam config env`

opam update
opam switch 4.02.3
opam pin add -y merlin git@github.com:the-lambda-church/merlin.git#87ea0e7998c04f16e4821676c27f19d3879dc2d1
opam pin add -y merlin_extend git@github.com:def-lkb/merlin-extend.git#ef634252a793542b05ec00a90f3c17de8fe0a357
opam pin add -y reason git@github.com:facebook/reason.git#b105ecd2b6728e3f62f11f82a2947c8ca2b59fed

```

Get Started Now
---------------
Download the up-to-date [docs](https://github.com/facebook/Reason/archive/docs.zip) which guide you through the basic syntax and toolchain features.

Contribute back to that documentation in the [`docs` branch](https://github.com/facebook/Reason/tree/docs).



Integrating with Existing Languages.
------------------------

 **`JavaScript` and `CommonJS`**:

Comming Soon.


**`clang`/`ocamlc`/`ocamlopt`**:

Though the easiest way to use `Reason` with `ocamlc/opt` is by running the compiler with the following flags:
```sh
# intf-suffix tells the compiler where to look for corresponding interface files
ocamlopt -pp refmt -intf-suffix rei -impl myFile.re
ocamlopt -pp refmt -intf myFile.rei
```

`Reason` can also be activated using `ocamlfind`.

``` sh
ocamlfind ocamlc -package reason ...
```

Convert Your Project from OCaml to Reason:
------------------------------------------------------------
`Reason` includes a program `refmt` which will parse and print and
convert various syntaxes. You can specify which syntax to parse, and
which to print via `-parse` and `-print` flags. For example,
you can convert your `ocaml` project to `Reason` by processing each file
with the command `refmt -parse ml -print re yourFile.ml`. Execute
`refmt -help` for more options.

### Removing `[@implicit_arity]`

The converted Reason code may attach `[@implicit_arity]` to constructors like `C 1 2 [@implicit_arity]`.
This is due to the fact that OCaml has the ambiguous syntax where a multi-arguments
constructor is expecting argument in a tuple form. So at parsing time we don't
know if `C (1, 2)` in OCaml should be translated to `C (1, 2)` or `C 1 2` in Reason.
By default, we will convert it to `C 1 2 [@implicit_arity]`, which tells the compiler
this can be either `C 1 2` or `C (1, 2)`.

To prevent `[@implicit_arity]` from being generated, one can supply `-assume-explicit-arity`
to `refmt`. This forces the formatter to generate `C 1 2` instead of `C 1 2 [@implicit_arity]`.

However, since `C 1 2` requires multiple arugments, it may fail the compilation if it is actually
a constructor with a single tuple as an arugment (e.g., `Some`).
We already have some internal exception rules to cover the common constructors who requires a single tuple
as argument so that they will be converted correctly (e.g., `Some (1, 2)` will be converted
to `Some (1, 2)` instead of `Some 1 2`, which doesn't compile).

To provide your own exception list, create a line-separated file that contains all constructors (without module prefix)
in your project that expects a single tuple as argument, and use `-heuristics-file <filename>`
to tell `refmt` that all constructors
listed in the file will be treated as constructor with a single tuple as argument:
```
> cat heuristics.txt
TupleConstructor
And
Or

> cat test.ml
type tm = TupleConstructor of (int * int) | MultiArgumentsConstructor of int * int

let x = TupleConstructor(1, 2)
let y = MultiArgumentsConstructor(1, 2)

module Test = struct
  type a = | And of (int * int) | Or of (int * int)
end;;

let a = Test.And (1, 2)
let b = Test.Or (1, 2)
let c = Some (1, 2)

> refmt -heuristics-file ./heuristics.txt -assume-explicit-arity -parse ml -print re test.ml
type tm = | TupleConstructor of (int, int) | MultiArgumentsConstructor of int int;

let x = TupleConstructor (1, 2);

let y = MultiArgumentsConstructor 1 2;

let module Test = {type a = | And of (int, int) | Or of (int, int);};

let a = Test.And (1, 2);

let b = Test.Or (1, 2);

let c = Some (1, 2);
```


Upgrading Existing Source Code After Changing Parse/Printing:
------------------------------------------------------------
To upgrade existing `Reason` code to a new version of the `Reason` syntax,
you should parse the code with the old version of the parser, and print it with
the new version of the printer. The easiest way to do this is to build the new
version of `Reason` locally, while the old one is pinned globally, but any way that you
have two versions installed locally will work.

The script `upgradeSyntax.sh` will execute reformattings for the entire
subdirectory repo across two separate builds of `Reason`. It expects you to
supply the path/command to the old build of Reason, the path/command for the
new build of Reason, and the print width (use `110`).  For example, if you had
an old version of `Reason` globally pinned as `refmt`, and a local build of
`Reason` at `~/github/Reason/refmt_impl.native`:

```sh
# Run from whichever directory you want to search
~/github/Reason/upgradeSyntax.sh refmt ~/github/Reason/refmt_impl.native 110
```

Deeper OCaml integration
---------------------------

If you are using `ocamlbuild`, add the following to your `_tags` file, but
this likely won't be enough because `ocamlc`/`ocamlopt` will need the
`-intf/-impl/-intf-suffix` flags:

```
<**/*.{re,.rei}>: package(reason), syntax(utf8)
```

Additionally, the OCaml compiler exports its internals, including the parser,
in a package `compiler-libs`. This allows us to parse *directly* into the
`OCaml` AST, instead of having an additional AST -> AST conversion step.

Findlib provides an interface that allows registering a preprocessor.
Additionally, it will pass all package include paths to such a preprocessor.

Developing:
-------------------------
See [DEVELOPING.md](./DEVELOPING.md).

Credit:
-------
The general structure of this project's organization and build/packaging
process was taken from @whitequark's m17n project, including parts of the
`README` that instruct how to use this with the OPAM toolchain.

License
-------

New content is licensed under the MIT license, works that are forked from other
projects are under their original licenses.
[MIT license](LICENSE.txt)

Editor plugins (which have also been forked) in the `editorSupport/` directory
include their own licenses.
