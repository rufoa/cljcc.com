---
layout: default
---

# cljcc

cljcc is a parser generator for Clojure. You can find it on [GitHub here](https://github.com/rufoa/cljcc). [![Build Status](https://travis-ci.org/rufoa/cljcc.png?branch=master)](https://travis-ci.org/rufoa/cljcc)

## Features

- Lexer:
    - supports string and regex tokens
    - uses lazy seqs, consuming as little input as possible
    - sensible error messages

- Parser:
    - supports LR(0) grammars
    - CST to AST transformations
    - sensible error messages
    - graphviz integration for debugging your grammar

## Usage

cljcc is available as a Maven artifact in [clojars](https://clojars.org/cljcc).

To use it in your leiningen project, add `[cljcc "0.1.0-SNAPSHOT"]` to the dependencies vector in your `project.clj` file.

The main namespace `cljcc` exports three functions: `make-lexer`, `make-parser` and `make-combined` (a convenience function which returns a combined lexer and parser).

```clojure
(ns my-project.example
    (:require [cljcc :refer [make-lexer make-parser make-combined]]))
```

These are the only functions you will need to use most of the time.

## Lexing

The `make-lexer` function takes a set of *token specs* (maps) and returns a lexing function. This function turns an input string into a lazy seq of tokens, which is suitable for use in the parsing stage.

Here is an example invocation:

```clojure
(def tokens
    #{  { :name :plus    :pattern "+" }
        { :name :mult    :pattern "*" }
        { :name :lparen  :pattern "(" }
        { :name :rparen  :pattern ")" }
        { :name :int     :pattern #"\d+" }
        { :ignore true   :pattern #"\s+" }})

(let [lex (make-lexer tokens)]
    (lex "(12 + 34) * 56 "))
```

### Token specs

A *token spec* is a map describing a particular token. It usually has a `:name`, and has a `:pattern`, which can be either a string or a regex.

The map may also have an `:ignore` entry, which when `true` results in the token being ignored. This is useful when working with languages which are whitespace-insensitive or can have inline comments in the source code.

### Priorities

By default, the lexer chooses the longest matching token. This allows you to have a token with `:pattern "for"` and another with `:pattern "foreach"`, for example.

In some circumstances, more than one token may match and also have the same length. You can add a `:priority` entry to the map to choose between them. The default priority is 0; the matching token with the highest priority wins.

In the following example, `:priority` is used to ensure the correct lexing of the `while` token in a string like `while (foobar == true)`. Without a `:priority`, cljcc might falsely assume `while` to be an identifier.

```clojure
(def tokens
    #{
        { :name :identifier  :pattern #"\w+"  :priority -1 }
        { :name :while       :pattern "while" }
        { :name :true        :pattern "true" }
        { :name :lparen      :pattern "(" }
        { :name :rparen      :pattern ")" }
        { :name :eq          :pattern "==" }
        { :ignore true       :pattern #"\s+"} })
```

### Valuation functions

Some regex tokens have a notion of 'value'.

If the token represents e.g. a hexadecimal literal, this would be its numeric value. If the token represents a quoted string literal, its 'value' would be the string itself (without the surrounding quotes and with any escaping removed).

The optional `:valuation-fn` entry can be used to capture this. It should be a unary function which takes the regex match result (the output of `re-find`) and returns the 'value'.

```clojure
{:name         :hex
 :pattern      #"0x([0-9a-fA-F]+)"
 :valuation-fn #(Integer/parseInt (second %) 16)}

{:name         :string
 :pattern      #"'((?:[^'\\]|\\.)*)'"
 :valuation-fn #(clojure.string/replace (second %) #"(?<!\\)(\\\\)*\\'" "$1'")}
```

### Lexer output

The output of the lexer is a lazy seq of _matched tokens_. A matched token is a map of token names and values, as well as source positions which can be used for debugging.

An end sentinel, the token named `:$`, is appended to the end of the seq.

For example:

```clojure
(def tokens
    #{  {:name :hex
         :pattern #"0x([0-9a-fA-F]+)"
         :valuation-fn #(Integer/parseInt (second %) 16)}
        {:name :plus    :pattern "+"}
        {:name :lparen  :pattern "("}
		{:name :rparen  :pattern ")"}
		{:ignore true   :pattern #"\s+"}})

(let [lex (make-lexer tokens)]
    (lex "(0x2a + 0x4d)"))
```

produces:

```clojure
(   {:token-name :lparen
     :consumed   "("
     :position   #cljcc.lexer.Position{:line 1, :char 1}}
     
    {:token-name :hex
     :consumed   "0x2a"
     :value      42
     :position   #cljcc.lexer.Position{:line 1, :char 2}}
     
    {:token-name :plus
     :consumed   "+"
     :position   #cljcc.lexer.Position{:line 1, :char 5}}
     
    {:token-name :hex
     :consumed   "0x4d"
     :value      77
     :position   #cljcc.lexer.Position{:line 1, :char 7}}
     
    {:token-name :rparen
     :consumed   ")"
     :position   #cljcc.lexer.Position{:line 1, :char 9}}
     
    {:token-name :$
     :position   #cljcc.lexer.Position{:line 1, :char 10}})
```

### Regex feature compatibility

cljcc uses [dk.brics.automaton](http://www.brics.dk/automaton/) internally to manipulate regex tokens. Although regular languages are fully supported — including the standard predefined character classes `\d \D \w \W \s \S` — more advanced features outside the scope of regular languages (e.g. word boundaries, lookaheads) are not.

Regex tokens can take advantage of Java SE 7's support for named capturing groups by using [named-re](https://github.com/rufoa/named-re).

## Parsing

lorem ipsum

### Production rules

lorem ipsum

## Debugging

lorem ipsum

## License

Copyright © 2014 [rufoa](https://github.com/rufoa/)

Distributed under the [Eclipse Public License 1.0](http://www.eclipse.org/legal/epl-v10.html), the same as Clojure.