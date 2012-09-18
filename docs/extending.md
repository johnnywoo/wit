# Extending WTF and technical details

## Writing your own commands

A general convention is to have SQL-like optional constructs in your arguments.
When you want to use an optional parameter, pair it with a keyword argument,
which you then can use to detect the presence of the parameter.
This removes any need in special characters while allowing easy parsing, by
both human eye and the command implementation:

    REPEAT 5 times " -> "
    REPEAT 80 chars "~="

Quotes can be used to distinguish keywords from values, of course:

    CONCAT glue ", " $a $b
    CONCAT "glue" $a

A command may be marked as 'no input', then it ignores any input but
preserves arguments (otherwise last argument may be removed and used as input).
Trying to give such command an inner block would be compile error;
pipelines are silently ignored. An example would be `file-syntax` command.

If you need to make a command name with multiple words, you should use hyphens.
Underscores can be used too, but that would introduce certain weirdness.
Camel case would be even worse because command statements use uppercased names.

Arguments and input may be expressions/code blocks.

Commands may have followup commands (ELSE for IF).

A command may be marked as pure (better to make it say per call) to indicate
that its output only depends on input and arguments. Then calling it with
literal arguments will make the compiler replace the call with its result.

??? use case: TRIM with inner block (can be computed if block starts/ends with literals)

## Definitions

The grammar can be found in `wtf.hatchet`.

??? more info here

 * Quotes syntax, lists of special symbols and \xAB possibilities
 * List of known single tags like br
 * Precise description of grammar, what is allowed where (expressions, tag args, cmd args, pipelines, etc)
 * Variable name syntax (cannot start on number, etc)

### Pseudo tags for input

Not all inputs have pseudo tags, that would be an overkill. HTML5 has 23 types!
`input` tag has `type="text"` by default. Pseudo tags are:

 * checkbox
 * radio
 * submit
 * hidden
 * password
 * file
