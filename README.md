# Wit is a template engine

Wit combines syntax elements of shell script, python, haml, css and sql.
It's a template language with static typing. It's awesome.

Pros:

 * Escaping problem (XSS) solved once and for all;
 * As little crap as possible;
 * Easy to write and read;
 * Can be compiled into any target language.

Cons:

 * There is a learning curve;
 * You don't want to write your blog posts in it: Wit is designed specifically
   for _templates_.

## Overview

Not so fast! Read the docs while the overview is (not) being written.

 * syntax.md: syntax of statements and expressions
 * commands.md: built-in library of commands
 * extending.md: recommendations on extending Wit and technical details
 * why.md: reasoning behind creating the language

## Why, oh God why?

Answer to your question lies within `docs/why.md`.

## TODO

show-syntax? generates an annotated template in HTML with colored escaping

What exactly do we need from context-awareness? (if we prohibit editing JS and CSS seems to be html-escaped)
How do we solve the issue with URLs?
