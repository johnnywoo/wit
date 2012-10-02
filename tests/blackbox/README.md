# Blackbox tests

Blackbox tests go here. These tests should be passed by any Wit implementation
using any compiler and so on. Basically these tests should validate that
an implementation of Wit takes template X and uses data Y with it to produce
correct output Z.

## Syntax of blackbox tests

A blackbox test is a text file `*.test` that consists of sections, much like
phpt tests:

 * `--INPUT--`
 * `--FILE--`
 * `--EXPECT--`
 * `--EXPECT-ERROR--`

`--INPUT--` is given in JSON format. This section may be omitted.

`--FILE--` is the body of the test template. If this section is not specified,
anything before first section name will be used as `--FILE--`.

`--EXPECT--` is the expected output that should be produced by applying given
input to given template. If this section is omitted, the test just validates
that the template can be compiled.

`--EXPECT-ERROR--` can be used to describe that an error is expected.

Any other sections are ignored. If you include a section twice, only last
definition will be used.

## Using blackbox tests

First, run `tests/blackbox2phpt` command, which will generate phpt tests out of
`.test` files. Then use Pyrus to run these tests.
