# Wit syntax

This is less of a tutorial and more of a syntax description with clarification
of tricky parts, so if you see something that wasn't explained yet,
just read on.

If you don't like reading walls of text, feel free to browse `tests/blackbox`
directory instead: there are tons of examples there.

## Types

Wit is a statically typed language. This means every value has a type.
Every variable has a type associated with it and you cannot assign values with
different types to the same variable. Commands can transform values
from one type to another.

A value can have a scalar type (like `text`, `html`) or be an array.

Scalars are basically numbers and strings. Scalar types are:

 * `number` — numeric type like float
 * `text` — default type for variables
 * `html` — can be made from other scalars by escaping html entities
 * `js` — no implicit conversion into it is possible
 * `json` — can be made from any type (including arrays) by json-encoding
 * `uri` — can be converted into `html` and `text` by url-encoding
 * `css` — ???

Arrays work like in PHP: array is an ordered list of unique scalar keys
which have values associated with them. Elements of an array can have different
types.

When a value is used in a place where different type is expected, an implicit
type conversion may occur. For example, outputting a plain text value in a HTML
block will result in text->html conversion, i.e. escaping. Same syntax checking
will be done in other contexts: if you try to insert a html value into a text
literal, Wit compiler will not let you do that because it doesn't know how to
turn html into text (it cannot be done without losing information).

## Variables: $foo

Variable names start with `$`. A variable can be embedded in text by simply
typing its name:

    Here is the value of foo: $foo

If you need to echo the dollar sign, use an expression with single quotes:

    Here is the value of {'$foo'}: $foo
    Wrap the var into an expression if the situation is {$a}mbiguous.
    You can use the $ as is if it does not make a variable name: $100.

Array element access is denoted with dots: `$company.name`. You can use other
variables and numbers the same way: `$list.0.$field`.

Undefined variables are treated as empty strings. Attempting to access
an array element on a scalar variable (`$string.element`) will be treated
like an undefined variable. There will be no errors if this occurs at runtime
(unless you have debug mode on, of course), but if the compiler can detect
usage of undefined variables, it will complain.

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
    Values can also be written literally: {'text'}
    Value expressions can also contain numeric operators: {1 + 2}

Expressions support pipelines. Those use commands:

    {'This is going to be uppercase' | uppercase}

You can use expressions inside of expressions:

    {"-- " | repeat {$nesting_level - 1} times}

An expression can be multiline.

    SET $beatles {array
      'John Lennon'
      'Ringo Starr'
      'George Harrison'
      'Paul McCartney'
    }

## Commands: if, for, ...

Command names are case-insensitive. You are, however, required to use
upper case names to indicate command statements (i.e. outside of expressions).
I use lowercase names in expressions to reduce visual noise.

    This is "Hello" with some modifications: {"Hello" | replace l L}
    The statement equivalent to that expression:
    REPLACE l L
      Hello
    Well, not exactly equivalent, but whitespace isn't visible, so it's OK!

Commands have space-separated arguments:

    SET $var $value

A command statement can have contents inside it, denoted by increased indent:

    SET $var
      Hello, world! This text is going to be assigned to the variable.
    This, however, is outside of the SET and therefore will become output.

Unlike expressions, command statements need to be placed on their own lines.
Command statements cannot be multiline, but expressions in them can be:

    FOR {array
      one two three four
    } as $name key $i
      Number $i has name $name.

Commands have arguments and input. Input can come from a pipeline (a-la stdin)
or in form of an inner block. If there is no pipeline and no inner block,
last argument will be removed and used as input instead; this provides uniform
interface to all commands, including user-defined. The difference between
an argument and input is the latter can be a code block, while arguments will
be computed before passing into the command.

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

Arguments, input and output of commands may be of any type.

    ul
      FOR {array 1 2 3}
        li > $it

## Tags: div, span, ...

Tag statements start with the tag name in lower case.

    table
      tr
        td

Tags can have space-separated attributes:

    a href=$profile_link
      View profile

Common attributes have short forms, as usual:

    a.class-name#id

Omit the tag name and you get a `div`:

    #overall
      .wrapper
        Content wrapped into two divs.

There are special things to make forms easier, namely `&` for `name`
attribute and pseudotags for common input types:

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

Script tag body is treated as a literal block:

    script
      var a = "It's not a <var> tag";

Style tag body ignores everything except variables:

    style
      td { /* no syntax error here */
        color: $color; /* variable value will be used */
      }

Same rules apply if you use the tag in its HTML form:

    <script type="text/javascript">
        var data = {a: 1, b: 2}; // not a Wit expression
    </script>

## String literals and quoting

Inside single quotes nothing is interpreted except `\\` and `\'`.

Double quotes can have special symbols:
`\t`, `\n`, `\r`, `\$`, `\{`, `\\` and `\"`.
Also, you can put variables and expressions inside double quotes.
Those expressions can even have double quotes inside them.

    a href="index.php?url=$link&title={"Note the quotes placement!" | uppercase}"

You can put literal newline characters into both types of quotes.

String literals in both types of quotes are of type `text`. You can prefix
a quoted string with a type name to change the type of a literal. For brevity
you can use prefix alias `h` for HTML. There should be nothing between
type name and the opening quote.

    SET $a "M&M's"
    {"Eat more <em>$a</em>"} produces Eat more &lt;em&gt;M&amp;M's&lt;/em&gt;
    {h"Eat more <em>$a</em>"} produces Eat more <em>M&amp;M's</em>

Note that `$a` was properly escaped both times, because its type is `text`.

The only place where a quoted string without prefix is not always `text` is
in tag attribute values. There the literal assumes the type of the attribute,
because having a compile error in `onclick="alert(1)"` is quite ridiculous.

Remember that it's that way only for literals!

    SET $a "alert('This is text!')"
    a onclick=$a // compile error, cannot make js out of text

    // what you should do is either mark the literal with correct type
    SET $a js"alert('This is text!')"
    // ... or force the type of the value
    a onclick={js $a} // this is not recommended, because it's unsafe

In tag definitions there are special rules:

    SET $v " v "

    a x=$v
    This produces '<a x=" v "></a>'

    a x="keke $v"
    This produces '<a x="keke  v "></a>'

    SET $args "a=<b> c=d"
    SET $t title
    a $args $t=$v
    This produces '<a a=&lt;b&gt; c=d title=" v "></a>'

## Literal blocks

To start a line of text with a word that looks like command/tag name,
you have a number of ways:

    I wanted to introduce myself, but the "I" was treated as a command name!
    {'I could have used an expression.'}
    {"Use double quotes if you want $variables and {'other expressions'}."}
    LITERAL
      Those expressions would also work for multiline text, where words can
      be mistaken for tags and there's no <be> in html5. It will look much
      cleaner if you use statements for literal blocks, though. The LITERAL
      command ignores any template syntax inside.
    p > TEXT
      You can put the command after a tag definition to denote a literal
      block and save space at the same time. The TEXT command allows
      $variables and {'expressions'} while disabling statements.

There will be much more markup than text, so it's justified to apply the
special syntax to the latter. Also we only allow latin letters in
tag/command names, so non-latin languages such as Russian get away scot-free!

    Another trick is to put an empty expression at the start of
    {} the line. This prevents it from being parsed as a html tag.

## Escaping and block syntax

Correct escaping is ensured with value types and _syntax_. Every part of
the template has a syntax. Default syntax is `html`.

Syntaxes and their behaviours are:

 * `html` — statements, expressions and variables are allowed; type is `html`
 * `text` — expressions, variables; type is `text`
 * `literal` — no template elements; type is `text`
 * `raw` — no template elements; type is `raw`
 * `js` — no template elements; type is `js`
 * `css` — variables; type is `css`

You can assign syntax to blocks:

    Mark the whole file as JavaScript:
    FILE-SYNTAX js

    Mark a block as css:
    SYNTAX css
      a {color: $var} /* the 'a' won't cause a syntax error */

You can mark tags with syntax:

    SYNTAX css in tag style

Tag attributes have types, e.g. `onclick` is of type `js`.
You can assign attribute types yourself:

    TYPE uri in tag a attribute href
    TYPE js in attribute onclick, onkeypress

Known URI attributes have type `uri`. Here's how it works:

    SET $username 'Carl Johnson'
    a href="search?username=$username"
    This produces:
    <a href="search?username=Carl%20Johnson"></a>

## Nesting rules

Recommended indent size is two spaces. Using tabs in whitespace-dependent
languages is generally considered as a full-blown march to the Dark side.

Inside a tag increased indent goes inside the tag:

    table
      tr
        td#cell1
      tr
        td#cell2

Inside a command increased indent means contents of the command;
same indent may be used as follow-up commands (IF/ELSE).

Whitespace-only lines are ignored so your editor can safely trim trailing
whitespace. Lines that only have a comment are also ignored.

If there is a plain `<tag>`, it receives its content by HTML rules,
not from indentation! This enables a useful trick:

    .outer-wrapper
      .inner-wrapper
        table.overall
          tr
            <td>
    This is still inside of the wrapper table, but now we don't have to
    {} indent our whole template by a half-screen.

    div
      There we go! The div is still inside of the table.

    The end tag must match the indent of opening one, though:
            </td>
          tr#another
            td
              Still the same table!

A more nice way to do wrappers is to define it as a template:

    TEMPLATE overall
      // I don't use inline nesting here so the template looks more menacing.
      .outer-wrapper
        .inner-wrapper
          table.overall
            tr
              td
                SLOT // this is where the wrapped content will be inserted
            tr#another
              td > TEXT
                One thing to remember here, though, is that defined templates
                have their own scope! You'll need to pass needed variables
                explicitly.
    WRAP overall
      Now this is wrapped into the whole thing above.

Indent whitespace is not used in output.

    pre
      This line will appear without any indent whatsoever.
        This one will be indented with two spaces.

## Comments

    // Inline comment
    /* Block comment (multiline) */

Comments do not appear in output.

Comments may start at empty lines, inside statements (tag or command)
and inside expressions.

Comment placement can be tricky sometimes.

    a href //> strong
      Not so strong now!

    // span > This text will not see the light of day.

    span // This is a comment.
    FOR $a // And this is a comment.
    span > // And even this is a comment.
    span > But this // is NOT! It starts in HTML, not inside a statement.

    table
    /* a block comment:
      a
    b (indents inside comments are ignored, of course)
      */ // it doesn't matter where it ends indent-wise
      tr // this is inside of table

    In expressions inline comments {//lalala} end at the expression boundary.
    Block comments, {however /*}, (lalala this is commented) */}, continue
    {} through everything until the comment is closed.
    Remember that /* this */ is not a comment! It cannot start in the body.

    {"
      String literals /* cannot */ have comments inside, but expressions
      embedded in them {//lalala} can.
    "}

Lines that only have comments on them don't count as block indents, just like
empty lines. You can comment a line at the very beginning, it won't close all
blocks above it.

    table
      tr
        td
    //  td
        td
      tr
        td
        // td
        td

### Typehinting

Block comments that start with two stars are special: they are used to describe
data structures. You don't have to do that, but it helps. Also the compiler can
do that for you. The spec looks like this:

    /**
     * $persons
     *   name
     *   position
     *   is_admin
     */

??? ...to be continued

## Operators

Value expressions can have operators in them.

### Numeric operators

Numeric operators are: + - * / %.

These operators are exclusively numeric. This means operands will be always
treated as numbers. There is no concatenation; use double quotes for it.

    SET $a "$head$tail"

Conversion of strings into numbers is implementation-defined. This is 
a template language, cut us some slack!

Adding arrays: if a variable is known to be an array at compile time,
the compiler will raise an error. Otherwise the array will be casted
to 0.

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




