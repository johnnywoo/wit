# Why Wit?

It's 2013 and yet I just wrote my own template engine. Isn't there one already?

Turns out, no.

## The rant

> The good thing about reinventing the wheel is that you can get a round one.
> Douglas Crockford

There are two basic ways in which a template engine can be rendered useless.
One: bloody escaping. Two: ugliness. Let's review some examples, shall we?

### PHP

For a PHP shop, the first obvious candidate for a template engine is PHP.
What with it being born as a template language and stuff?

PHP does not have any mechanism for doing escaping automatically. You're
reduced to either doing it manually on every echo (and everybody always forgets
to do it), or to escape everything before the template receives it, so you
cannot properly pass values to functions.

The ugliness speaks for itself:

    PHP:
    <? foreach($payments as $payment) { ?>
        <tr>
            <td>
                <a href="?menu=pay&paymenu=pay_firm_edit&id=<?=hsc($payment['order_id'])?>"><?=hsc($payment['order_name'])?></a>
                <?=hsc($payment['order_number'])?>
            </td>
            <td>
                <?=hsc(dict('bonuses_source_names', $payment['source']))?>, payment <?=hsc($payment['pay_index'])?>
            </td>
            <td align="right"><?=num_tbl_fmt($payment['amount'])?></td>
        </tr>
    <? } ?>

    Wit:
    FOR $payments as $payment > tr
        td
            a href="?menu=pay&paymenu=pay_firm_edit&id={$payment.order_id}"
                $payment.order_name
            $payment.order_number
        td
            {dict bonuses_source_names $payment.source}, payment $payment.pay_index
        td align="right"
            NUM_TBL_FMT $payment.amount

Did you notice that `num_tbl_fmt` call was not escaped? Well, if you did,
then you should know that's _correct_, because it returns HTML. Fooled ya, huh?

### Smarty

Okay, so why not use the first thing that jumps at you the moment you say
'template engine for PHP'? Well, believe it or not, it's actually harder to
use than plain PHP, even though it's written in it.

The problem with Smarty is escaping again, which is as crooked as it can be.
You can either turn automatic escaping off and write all escapes manually,
or you can turn automatic escaping on and voilà: now you don't know what is
being escaped and where, because Smarty escapes values on _usage_, not on
output. So you try to pass a value into a custom function and an escaped
version may come through. Or not. Don't even get me started on array issues.

### Twig

Twig has a pretty website, maybe it's the bee's knees? It even has a _why twig_
section on the main page!

    Twig:
    {% for user in users %}
        * {{ user.name }}
    {% else %}
        No user have been found.
    {% endfor %}

    Wit:
    FOR $users as $user
        * $user.name
    ELSE
        No user have been found.

Note how the Wit template isn't littered with distracting characters.

The other thing is still the escaping done wrong. The manual calls autoescaping
an _unique_ feature; does that ring any bells for you? It's unique alright:
`{{ "<a>" }}` produces `<a>`, but `{{ var_with_that }}` produces `&lt;a&gt;`.
Good luck guessing what an expression will produce.

### XSLT

Now you probably think: if you love the stinkin' escaping so much, why don't
you use XSLT? It got the escaping right!

The main answer is blindingly obvious, of course. The kind of 'make me unsee'
obvious.

    XSLT:
    <select class="form" name="parent">
        <option>
            <xsl:attribute name="value"><xsl:value-of select="/root/rootId" /></xsl:attribute>
            <xsl:text></xsl:text>
        </option>
        <xsl:for-each select="/root/groups/item">
            <xsl:if test="canParent='true'">
                <option>
                    <xsl:if test="active = 'false'" >
                        <xsl:attribute name="class">disabled-item</xsl:attribute>
                    </xsl:if>
                    <xsl:attribute name="value"><xsl:value-of select="id" /></xsl:attribute>
                    <xsl:value-of select="fullTitle"  />
                </option>
            </xsl:if>
        </xsl:for-each>
    </select>

    Wit:
    select&parent.form
        option value=$rootId
        FOR $groups > IF $it.canParent
            option value=$it.id {if {!$it.active} class=disabled-item}
                $it.fullTitle

Other important problem with using XSLT is XML, which you have to generate.
XML generation has a number of problems, like when your data has keys like
'8march' which are not valid XML tag names. Things get bloated very quickly.

### TAL

Template Attribute Language is a smart concept that tries to do away with bloat
of XSLT while keeping the good parts.

    TAL:
    <div class="item" tal:repeat="item itemsArray">
        <span tal:condition="item/hasDate" tal:replace="item/getDate"/>
        <a href="${item/getUrl}" tal:content="item/getTitle"/>
        <p tal:content="value/getContent"/>
    </div>

    Wit:
    FOR $itemsArray as $item > div.item
        IF $item.hasDate > span > $item.getDate
        a href=$item.getUrl > $item.getTitle
        p > $value.getContent

The problem is you cannot make the bloat go away if you use XML syntax, because
XML is a _markup_ language, not a template language. XML is good when you have
large bodies of text with an occasional `<em>` in them, not heaps of statements
with very little literal text.

    PHPTAL:
    <span tal:replace="string:This is a $var value."/>

    Wit:
    This is a $var value.

### Mustache

There is a class of template languages that declare themselves logic-less.
Mustache seems to be a prominent example of this kind of thinking. Here, I'll
quote their doc: "We call it "logic-less" because there are no if statements,
else clauses, or for loops." If you actually read the manual, though, you will
soon discover that you should use blocks to make if statements, else clauses
and for loops. This is because a template without logic is a plain gorram
HTML file.

The whole idea behind template engines is to separate data from presentation,
so you need to put presentation logic into the template! If you don't want to
do that, you will do it anyway, only instead of one file you will have two:
a file that has a number of labeled strings, and a code block that's actually
doing the templating.

Consider this:

    Mustache:
    <h2>Names</h2>
    {{#names}}
      <strong>{{name}}</strong>
    {{/names}}

    Wit:
    h2 > Names
    FOR $names > strong > $it.name

Now you want to show the number of entries. How do you do that in Mustache?
Well, you make the outer code provide you with `count`. In all places that
template is used. Separation of concerns, my ass.

    Wit:
    h2 > Names
    FOR $names > strong > $it.name
    Total number of names: {count $names}.

### HAML

HAML is concise, hip and smart. So, why not HAML?

    HAML:
    %a(title=@title class="widget_#{@widget.number}") Stuff

    Wit:
    a title=$title class="widget_{$widget.number}" > Stuff

Well, if you like percent marks and parens and marks and parens, the syntax
can be tolerable. Especially since half of it is Ruby anyway. So we cannot
use it with PHP without writing a port. By the way, I googled 'php haml' and
the first project Google found for me had a glaring XSS issue right in their
first example rendering.

Also I could not find any way to reduce nesting (and HTML templates tend to
have _lots_ of nesting). In Wit we have `>` and native tag nesting for that.

The important issue, though, is 'escape html on echo' yet again.

### Latte

Latte has context-aware escaping, which is a huge step forward from the
'html escape on echo' approach. It may be unobvious, but it's unobvious
in a safe way: in most of common cases, a misunderstanding will get you a
properly escaped wrong value instead of a syntax error/XSS.

    Latte:
    {var $movies = 'Amarcord & 8 1/2'}
    <script>var movies = {$movies};</script>
    <p onclick="alert({$movies})">{$movies}</p>

    Wit:
    SET $movies 'Amarcord & 8 1/2'
    JSON-SCRIPT movies $movies
    p onclick="alert($movies)" > $movies

The difference here is Wit doesn't allow constructed JavaScript blocks.

??? or does it? to think it through, we need.

The reason for such a setback is simple: `{$movies}` can be valid JS.
Same with any kind of sane expression delimiter and naming scheme. A lot of
people actually uses `$` prefix as a Hungarian notation for jQuery objects.
Attribute literals are usually small enough to justify escaping non-Wit
`$` signs for the sake of consistency.

Unfortunately, you cannot alter the context detection (e.g. if you want to use
a `data-json` attribute) without digging deep into the Latte parser. That's not
a problem with the language, of course.

Also there is no way to specify that your helper returns HTML. If you want to
make an `nl2br` helper, you have to manually specify raw output mode every time
you use it.

## The way of the Force

A good template language should focus on three things:

 1. Escaping
 2. Readability
 3. Consistency

If you get these right, all that's left is much simpler: make a compiler,
arrange a good standard library and you're done.

### Escaping

To solve the escaping problem, we must use the Force. No, really.
Situations to consider are:

    {$var} — this should be escaped.
    CMD $var — the command should receive the value untouched.
    {$var | nl2br} — this should work properly.
    span title=$a — a space character can break HTML here.
    div onclick=$var — lots of things can break JS here.
    a href="/items/$id" — it would be really great to have $id urlencoded.

To solve this problem the proper way we'll use a novel concept from before
I was born: weak static typing. What this means is all incoming data is
marked to be plain text, and every command is marked: what can go in and what
comes out. These types can have relations between them, like plain text can be
automatically converted to html by escaping it (but not the other way around!
It would be a lossy conversion, so the user must explicitly do it herself).
This way we can check all operations at compile time to ensure the types all
match up (and insert conversions where appropriate) and nobody's trying to
inject a html string into an URL.

Our escaping solution still has one breach: dynamic template reuse (when you
use different templates depending on variable values) cannot be checked
statically without major pain. We can solve it by, well, removing dynamic
reuse. Specifically, we restrict dynamic includes by a name mask, so compiler
can check all possible candidates while you still can make the included name
depend on a variable.

With static typing we need to have some sort of an input spec to know what
data comes into the template (default is plain text, but you still need a way
to mark otherwise without having to cast contents of arrays everywhere).
This would also be quite useful for an IDE plugin, should one be written.
The engine could generate such a spec when a template is being rendered with
debug mode on, so you only need to change non-default types.

### Readability

HTML has very low readability if used for anything other than 100-page reviews
of Mac OS X. In case of a template language, which adds its own junk into the
mess, HTML is a no-go right from the start. So, we need to remove as much HTML
syntax as possible. Therefore, we need to use indent-based nesting and
HAML-like tags.

HTML can generate a very deep nesting though. We'll take `>` selector from CSS
to describe inline nesting (also called block expansion). Also we'll allow
HTML tags in their natural form to take advantage from XML nesting if needed
and for that rare inline `<em>`.

Next we need to decide which special characters to use to wrap our template
commands into. The best kind of delimiter, frankly, is no delimiter at all.
Templates are made of 50% html tags, 40% template commands and 10% text with
variables in it, so which of those should require delimiters? Yup. We already
have whitespace as a functional part of the syntax, so there you go.

How do we tell commands and tags apart without unleashing an army of ugly
characters on them? Uppercase and lowercase. We could, of course, put them in
the same namespace, but that would be harder to extend: introducing new
commands could break existing code (??? clarify or gtfo).

Next up: command arguments. Here you can't get more readable than shell with
its space-separated keywords. Remember, template commands are tiny one-liners.
We will take optional argument syntax from SQL though, because it looks good.
Combining commands together with no braces and commas? Pipelines, of course.

Tag argument syntax can be taken straight from HTML.

We still need something for inline expressions, because those are inevitable
when working with text. Every programmer out there is highly skilled when it
comes to typing curly braces, so the choice is kinda obvious. We must make
a sacrifice here, though: curlies mean we must treat JS code as literal.
Luckily JS is a programming language and doesn't need to be generated!
JSON variable passing should be allowed, of course, but it has nothing to do
with syntax.

Inline expressions should be made embeddable into string literals and other
expressions, just like in any programming language. To do that and still not
require commas and braces everywhere, we should embed expressions with curlies,
just like when you insert an expression into HTML body.

Coincidentally, readable syntax which has little to no weird characters is also
very easy and pleasant to write.

### Consistency

The syntax of template language should be simple and easy to understand.
For example, you can let your variables be named whatever they want and then
use _special_ syntax for embedding (which goes on a lot in templates), or
you can make variable names start with `$` and then embedding a variable
is simply typing its _name_ into text.

More importantly, everything that can be made unified should be unified.
An `if` statement does not deserve any more special treatment than your
third-party custom command for drawing lolcats. This also means that having
macros, filters, blocks and helpers as separate entities is meaningless.
Everything these things do is taking input and producing output, so they are
all commands. Shell has it like this since forever, but somehow nobody notices.

It would be really nice if the language would not be defined in terms of
the target language of the compiler. This way we could use the same template
for frontend and backend. That's easy if your backend is in JS and not so much
otherwise.
