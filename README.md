# dart_repl

## About

A proof of concept [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) environment for Dart.

### Example 1

[![asciicast](https://asciinema.org/a/1qkiplqr2pp99zmgjyo2rifbq.png)](https://asciinema.org/a/1qkiplqr2pp99zmgjyo2rifbq)

### Example 2

[![asciicast](https://asciinema.org/a/2wxg2qpnlcaw4dpoo6o2c705s.png)](https://asciinema.org/a/2wxg2qpnlcaw4dpoo6o2c705s)

### Further work

See the [Dart REPL Directions brain-dump](https://docs.google.com/document/d/1gDkF1meFpsQO_X_SCoAxdsdCNVhPiAz4HLyvvzOXWKU/edit?usp=sharing) for possible ideas and directions.

## Usage

### `pub global run`

You can install and setup `dart_repl` using:

    pub global activate dart_repl

To run it, simply execute:

    pub global run dart_repl

If you run it from a directory that contains a Dart package (it needs a .packages file), it will load all
dependencies automatically and allow you to import libraries adhoc from the command-line:

    pub global run dart_repl --adhoc-import package:built_collection/built_collection.dart

(if your package depends on built_collection).

Alternatively, you can use the `import()` builtin command instead of the commmand-line flag `--adhoc-import`:

    $ pub global run dart_repl
    
    >>> import('package:built_collection/built_collection.dart');
    
This is the preferred way of running dart_repl as it requires no additional setup.

### From another package

You can add a `dev_dependency:` to your `pubspec.yaml`:

```
dev_dependencies:
  dart_repl:
  [...]
```

You can then run the REPL with:

    pub run dart_repl

It will automatically resolve all additional adhoc imports against the dependencies of your package: 

    pub run dart_repl --adhoc-import package:built_collection/built_collection.dart

### From a checkout

From the command-line

    dart bin/dart_repl.dart

To import additional libraries, you need to specify a package directory (--package-dir) to allow 
it to resolve dependencies:

    dart bin/dart_repl.dart --package-dir ~/git/built_collection.dart/ --adhoc-import lib/built_collection.dart

## Features requests and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/BlackHC/dart_repl/issues

## Appendix

### TODOs

[X] provide a back channel for the sandbox to change its own package config.
[] find a way to unexport a library (can we enumerate all the symbols from a library and hide them?)

## Changes and open questions

### Variable shadowing

Variables need to be declared with `var` or `final` now.
Each top level declaration spawns a new compilation unit. All compilation units are chained
together. This means that you can freely redeclare variables. They will shadow each other.
This can lead to unintended consequences though:

```
>>> var a = 1
>>> void ip() { print(a++); }
>>> ip()
1
>>> ip()
2
>>> var a = 1;
>>> ip()
3
>>> a
1
```

### Old Scope behavior

The old behavior (non-shadowed, undeclared variables) is available using the scratch Scope.

```
>>> scratch.a = 1
1
>>> void ip() { print(++scratch.a); }
>>> ip()
2
>>> ip()
3
>>> scratch.a = 1
1
>>> ip()
2
```

#### Why can't I keep this default?

The Scope behavior is ideal, however, I don't know how to lift it into the global namespace.
Before I was evaluating every expression and statement within the Scope, so you could access
all its fields without qualification. However, with top-level declaration, this is not
possible anymore. This would mean having different semantics:

```
>>> a = 1
1
```

but

```
>>> class X { void ip() { print(++scratch.a); } }
```

A solution would be to generate a global scope file that is constantly reloaded.
This seems infeasible for more complex inputs and for redefinitions.

### Export not import!

Because each top-level decl is its own compilation unit, `import`s only work within the
same cell/input.

```
>>> import 'dart:io'; get pwd => Directory.current;
>>> pwd
Directory: '/Users/blackhc/IdeaProjects/dart_repl'
>>> Directory.current
Unhandled exception:
NoSuchMethodError: No top-level getter 'Directory' declared.
```

To make a library available to following cells, you have to use `export`.

```
>>>  export 'dart:io'
>>> Directory.current
Directory: '/Users/blackhc/IdeaProjects/dart_repl'
```

