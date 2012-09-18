# WTF Syntax

## Variables: $foo

In html and tag/command attributes you can output/use variables.
Variable names start with $:

    Here is the value of foo: $foo

If you need to echo the dollar sign, use an expression with single quotes:

    Here is the value of {'$foo'}: $foo
    You can use the $ as is if does not make a variable name: $100.

There are two types of data: scalars and arrays.

Scalars are basically strings. There is no separate type for numbers.

Arrays work like in PHP: array is an ordered list of unique scalar keys
which have values associated with them. Array element access is denoted
with dots: `$companies.0.name`.

## Expressions: {}

Expressions are denoted by curly braces. Expressions can be placed anywhere
in the template body, inside other expressions, inside double quotes and
generally everywhere else. In template body expressions output their values;
otherwise the value is used in regards with context.

There are two kinds of expressions: command expressions and values.

Command expressions are an inline form of using commands.
They start with a command name:

    Today is {date d.m.Y}, isn't that great?

Value expressions start with a value instead of a command:

    This is an explicit output of a variable: {$var}
    A value can also be written literally: {'text'}
    It can also contain numeric operators: {1 + 2}

Expressions support pipelines. Those use commands:

    {'This is going to be uppercase' | upper}

You can use expressions inside of expressions:

    {"-- " | repeat {$nesting_level - 1} times}

An expression can be multiline. There should be no whitespace after `{`.

    SET $beatles {make-list
        'John Lennon'
        'Ringo Starr'
        'George Harrison'
        'Paul McCartney'
    }

## Commands: if, for, ...

Command names are case-insensitive. You are, however, required to use
uppercase names to indicate command statements (i.e. outside of expressions).
I use lowercase names in expressions to reduce visual noise.

    This is "Hello" in base64-encoded form: {"Hello" | encode base64}
    The statement equivalent to that expression:
    ENCODE base64
        Hello
    Well, not exactly equivalent, but whitespace isn't visible, so it's OK!

Commands have space-separated arguments:

    SET $var $value

A command statement can have contents inside it, denoted by increased indent:

    ENCODE base64
        Hello, world! This text is going to be encoded.
    This, however, is outside of the SET and therefore will not be encoded.

Unlike expressions, command statements need to be placed on their own lines.
Command statements cannot be multiline, but expressions in them can be:

    FOR {make-list
        one two three four
    } as $name key $i
        Number $i has name $name.

Commands have arguments and input. Input (a-la stdin) can come from a pipeline
or in form of an inner block. If there is no pipeline and no inner block,
last argument will be removed and used as input instead; this provides uniform
interface to all commands, including user-defined.

    Inner block input:
    REPEAT 5 times
        Input

    Pipelined input:
    {"Input" | repeat 5 times}

    Input from last argument:
    {repeat 5 times Input}
    Also:
    REPEAT 5 times Input
    ...without inner block.

Arguments, input and output of commands may be scalar or array.

    SET $list {make-list one two three}
    ul
        FOR $list
            li > $it

## Tags: div, span, ...

Tags have their attributes much like commands:

    a href=$profile_link
        View profile

Common attributes have short forms, as usual:

    a.class-name#id

There are special things to make forms easier, namely &-syntax for 'name'
attribute and pseudotags for input types:

    checkbox&person[is_vip]
    is equivalent to
    input type=checkbox name=person[is_vip]

Tags can be written in a short nested form:

    table
    	thead > tr
    		th > Color name
    		th > Hex code

Actually, you can mix commands and tags like this:

    ul > FOR $list key $i > li
        item #$i: $it
    It's equivalent to:
    ul
        FOR $list key $i
            li
                item #$i: $it

??? What about this?

    ul
    > FOR $list key $i
    Still inside for
    > li
    Still inside li
    > item #$i: $it

Script tag body is treated as a literal block:

    script
        var a = "It's not a <var> tag"

Style tag body ignores tag syntax, while commands, variables and expressions
continue working:

    style
        td { /* no syntax error here */
        	color: $color; /* variable value will be used */
        }

??? table {blah: syntax error. no such command}

## Quoting

Inside single quotes nothing is interpreted except `\\` and `\'`.

Double quotes can have special symbols like `\t` and `\n`.
Also, you can put variables and expressions inside double quotes.
Those expressions can even have double quotes inside them.

    a href="index.php?url=$link&title={"Note the quotes placement!" | lowercase}"

In tag definitions there are special rules:

    SET $v " v "
    a x=$v
        this produces '<a x=" v "></a>'
    a x="keke $v"
        this produces '<a x="keke  v "></a>'
    SET $args "a=<b> c=d"
    SET $t title
    a $args $t=$v
        this produces '<a a=&lt;b&gt; c=d title=" v "></a>'

You can put newline characters into both types of quotes.

### Literal blocks

To start a line of text with words that look like commands/tags, simply
enclose the text in a literal expression:

    I wanted to introduce myself, but the "I" was treated as a command name!
    {'I should have used an expression.'}
    {"Use double quotes if you want $variables and {'other expressions'}."}
    {'
        It also works for blocks of natural language text, where words can be
        mistaken for tags and there's no <mistaken> in html5.
    '}
    p > {'
        You can put the expression after a tag definition to denote a literal
        block and save space at the same time.
    '}

There will be much more markup than text, so it's justified to apply the
special syntax to the latter. Also we only allow latin letters in tags,
so non-latin languages such as Russian get away scot-free!

## Operators

### Numeric operators

In templates, we only need the basic operators: + - * / %.
Minus and plus should work in unary and binary forms.

Conversion into numbers:

??? depends on the compiler! bwahahaha!

### Boolean operators

Also comparison operators would be nice: == != > >= < <=.
Also boolean operators and grouping: () ! && ||
??? Maybe natural order here? Lots of complexity won't work for sorting!

Conversion into boolean:

 * nonexistent var = false
 * "" = false
 * numeric value is 0 = false
 * empty array = false

Anything else is true.

### Ternary operator

Come to think of it, let's have a ternary: a ? b : c

## Escaping

Escaping and allowed syntax elements are determined by _syntax_.

 * html — the default syntax, everything works here; output is html-escaped
 * text — tags and commands don't work, expressions and variables do;
   output is not escaped
 * css — only variables; output is not escaped ??? how do you even?
 * raw — nothing works; output is not escaped
 * js — nothing works; output is json-encoded
 * uri — variables and espressions; output is url-encoded ???

You can assign syntax:

    Mark the file as JavaScript:
    FILE-SYNTAX js

    Mark a block as css:
    SYNTAX css
        a {color: $var} /* the 'a' won't cause a syntax error */
    p > Back to HTML again

    Mark a certain tag/attribute:
    SYNTAX css in tag style
    SYNTAX uri in tag a attribute href
    SYNTAX js in attribute onclick, onkeypress

While 'js' syntax is marked as 'nothing works' just as the 'raw' syntax,
there is an important difference of

    SYNTAX json in attribute data-item
    div data-item=$data
    It will produce something like this:
    <div data-item="{&quot;key&quot;:&quot;value&quot;}"></div>

    Let's try that with other syntax:
    SET $a '1 < 2'
    SYNTAX raw in attribute data-raw
    div data-raw=$a
    It will produce something like this:
    <div data-raw="1 < 2"></div>

??? I'll get back to you on that one.

Escaping is used at output, in other places the values are passed
without modification.

Literals are text: `{'&'}` produces `&amp;`.

??? to generate HTML and then echo it, you do ???

??? examples!

??? maybe we need to taint the values? what about commands that
generate html?

## Nesting rules

Inside a tag increased indent goes inside the tag:

    table
        tr
            td#cell1
        tr
            td#cell2

Inside a command increased indent means contents of the command;
same indent may be used as follow-up commands (IF/ELSE).

Whitespace-only lines are ignored so your editor can safely trim trailing whitespace.

If there is a plain <tag>, it receives its content by HTML rules,
not from indentation! This enables a useful trick to lower nesting:

    table.overall
       thead
           tr
               <td>
        This is still inside of the wrapper table, but now we don't have to
        = indent our whole template by a half-screen.

    div
        There we go! The div is still inside of the table.

    The end tag must match the indent of opening one, though:
                </td>
           tr#another > td
               Still the same table!

The HTML is required to be valid nesting-wise (you can write <br>,
but non-html5 single tags must be <closed />).

## Comments

    // Inline comment
    /* Multiline comment */

Comments do not appear in output.
