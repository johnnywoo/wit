# Library of Wit commands

Remember that last argument is input and is interchangeable with an inner block
or an incoming pipeline.

Special placeholders used below:

 * `<variable>` means a variable name with the `$` sign, e.g. `$foo` or `$a.b.c`.
 * `<condition>` means a value expression without curly braces, e.g. `1 + 2`.
   Expressions allow nesting, so `{1 + 2}` is also a valid `<condition>`.

Square brackets define optional elements.

## Control flow

### if

    IF <condition> <input>
    ELSE [if <condition>] <input>

We need support for inline {if cond input}

### for

    FOR <array> [as <variable>] [key <variable>] [glue <value>] <input>

Default variable name for value is `$it`. There is no default name for key
variable.

Glue will be output between iterations.

    FOR $a
        table
            ITEM > tr
                td
    GLUE
        tr > td > hr
    ELSE
        tr > td > Nope

    FOR $a
        tr
            td
    WRAP
        table > SLOT

### ??? list

	LIST <array> <input>
	ELSE <input>

### ??? tree

## Template inheritance and reuse

### template

	TEMPLATE <name> <input>

Template name is any string.

### include

	INCLUDE file <filename>
	INCLUDE <template-name>

Includes given template at current point. The included template uses
the same scope as the `include` command.

Does not allow input.

### exec

    EXEC file <filename> {<variable> <value>}
    EXEC <template-name> {<variable> <value>}

Executes given template at current point. The executed template uses
its own scope. Values can be passed into the template via arguments.

Does not allow input.

    TEMPLATE person
        h1 > $name
        $position

    EXEC person name="John" $position

??? up

??? down

    SOMETHING template-name
      This will be wrapped into a template (using default slot).

### extend

	EXTEND <filename> [into <slot-name>]

Executes given template inserting current one into `slot` there.
If `<slot-name>` is not given, a nameless `slot` would be used.

Does not allow input.

### slot

	SLOT [<name>]

Used with `extends` and `wrap`.

## Syntax-related

### syntax

    SYNTAX <syntax> <input>

Defines syntax for a block/value.

    SYNTAX <syntax> in tag <tag> [attribute <attribute>]

Marks a tag as having certain syntax inside. If attributes are supplied,
marks values of those attributes instead.

    SYNTAX <syntax> in attribute <attribute>

Marks an attribute of any tag as having certain syntax.

???

    SYNTAX css +statements

### text
### literal

### file-syntax

    FILE-SYNTAX <syntax>

Defines syntax for current template file. Does not allow input.

??? muat be first in the file

## Data-related

### set

    SET <variable> [and echo] [lines of] <value>
    SET <variable> (+= | -= | *= | /= | %=) <value>

Assigns a value to a variable. If `lines of` is given, splits input into lines
and collects trimmed strings into an array.

???

    SET <variable> as <type>
      kek

### json

    JSON <value>

Produces json-encoded version of the data. Example:

    div data-attr={json $data}

In this particular example you would be better off using `syntax` command,
of course.

### json-var

    JSON-VAR <varname> <value>

Produces a script tag with the value:

    <script type="text/javascript">var varname = {...}</script>

## Text manipulation

### repeat

    REPEAT <num> times
    REPEAT <num> chars

### trim

    TRIM [left | right] $string

### ...

uppercase, lowercase, ucfirst
date

## Array manipulation

### sort

    SORT [by <field>] [desc] <array>

Returns sorted array. Default order is ascending.

If `by <field>` is given, the array is supposed to be a rowset (array of
arrays with similar objects). E.g. if you have a list of persons, you might
want to do something like this:

    ul > FOR {$persons | sort by name} as $person
      li > $person.name works as a $person.position

### keys

    KEYS <array>

Returns a list of keys of the array. Keys of the new array will be numeric
starting from 0.

??? Using inner block input will result in an array

### array

    ARRAY {<item>}

Returns an array with arguments. Keys of the array will be numeric starting
from 0.

Does not allow input.

    table#calendar
      thead > tr > FOR {array Mo Tu We Th Fr Sa Su} > th $it

### count

    COUNT <array>

filter
??? map
head, tail
array operators?
