# Blackbox tests

Blackbox tests go here. These tests should be passed by any WTF implementation
using any compiler and so on. Basically these tests should validate that
an implementation of WTF takes template X and uses data Y with it to produce
correct output Z.

## Syntax of blackbox tests

A blackbox test is a text file `*.test` that consists of three sections,
much alike phpt tests:

 * `--INPUT--`
 * `--FILE--`
 * `--EXPECT--`

`--INPUT--` is given in JSON format. This section may be omitted.

`--FILE--` is the body of the test template. If there are no section headers,
the whole test is treated as `--FILE--`.

`--EXPECT--` is the expected output that should be produced by applying given
input to given template. If this section is omitted, the test just validates
that data can be applied to the template without errors.

## Using blackbox tests

First, run `tests/blackbox2phpt` command, which will generate phpt tests out of
`.test` files. Then use Pyrus to run these tests.
