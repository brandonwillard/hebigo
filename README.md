# Hebigo
蛇語(heh-bee-go): Snake-speak.
Hebigo is an indentation-based Hissp skin designed to resemble Python.

Hebigo is still in the prototyping phase. See the native tests for example Hebigo code.

Hebigo keeps Python's expressions as *bracketed expressions*,
but completely replaces Python's *statements* with *hotword expressions*,
which have Hissp's literals, semantics, and macros.

## Bracketed Expressions
Bracketed expression are called that because they must be "bracketed"
somehow in order to be distinguishable from the hotword expressions.
Parentheses will always work, but `[]` or `{}` are sufficient.
Quotation marks also work, even with prefixes, like `b''` or `f""""""`, etc.

Bracketed expressions are mainly used for infix operators, simple literals, and f-strings
(things that might be awkward as hotword expressions),
but any Python expression will work,
even more complex ones like nested comprehensions or chained method calls.
It's best to keep these simple though.
Because you can't use macros in them.

## Hotword Expressions
Hotword expressions are called expressions because they evaluate to a value,
but they resemble Python's statements in form, like
```
word:
   block1
   subword:
       subblock
   block2
```
etc.

The rules are as follows.

1. A word ending in a `:` is a "hotword", that is, a function or macro invocation that can take arguments.
```
hotword: not_hotword
```

2. A hotword with no whitespace after its colon is *unary*. Otherwise it's *multiary*.
```
unary:arg
multiary0:
multiary1: arg
multiary2: a b
multiary3: a b c
```

3. Multiary hotwords take all remaning arguments in a line.
```
hotword: arg1 arg2 arg3: a:0 b: 1 2
```
Parsed like
```
hotword(arg1, arg2, arg3(a(0), b(1, 2)))
```

4. The first (multiary) hotword of the line gets the arguments in the indented block for that line (if any).
```
multiary: a b c
    d e f
foo unary:gets_block: gets_line: 1 2
    a b
    c d
```
Parsed like
```
multiary(a, b, c, d, e, f)
foo
unary(gets_block(gets_line(1, 2), a, b, c, d))
```
Another way to think of it is that a unary applied to another hotword creates a *compound hotword*, which is a composition of the functions.
In the example above, `foo` is not a hotword (no colon),
and the compound hotword `unary:gets_block:` is the first hotword of the line,
so it gets the indented block below the line.

5. The special hotword `pass:` uses its first argument for the invocation.
This allows you to invoke things that are not words, like lambda expressions.
```
pass: foo a b
pass: (lambda *a: a) 1 2 3
```
Parsed like
```
foo(a, b)
(lambda *a: a)(1, 2, 3)
```
### Style
These indentation rules were designed to resemble Python and make editing easier with a basic editor than for s-expressions.
As a matter of style, arguments should be passed in one of three forms, which should not be mixed for function calls.
```
linear: a b c d
linear_block:
    a b c d
block:
    a
    b
    c
    d
# What NOT to do, although it compiles fine.
bad_mix: a
   b c
   d
```
compare that to the same layout for Python invocations.
```
linear(a, b, c, d)
linear_block(
    a, b, c, d
)
block(
    a,
    b,
    c,
    d,
)
# What NOT to do.
bad_mix(a,
    b, c
    d
)  # PEP 8 that. Please.
```
The above is for function calls only.
Macro invocations are not exactly bound by these three layout styles,
and may instead have other documented preferred layouts.

You should group arguments using whitespace when it makes sense to do so.
Anywhere you'd use a comma (or newline) in Clojure, you add an extra space or newline.
This usually after the `:` in function invocations or parameter tuples,
where the arguments are implicitly paired.
```
linear: x : a b  c d  # Note extra space between b and c.
linear_block:
    x : a b  c d
block:
    x
    : a b  # Newline also works.
    c d
```
### Literals
Literals are mostly the same as Lissp for hotword expressions
(and exactly like Python in bracketed expressions).

Hebigo does not munge symbols like Lissp does.
Qualified symbols (like ``builtins..print``) are allowed,
but not in bracketed expressions, which must be pure Python.

*Control words* are words that start with a `:`.
These are not allowed in bracketed expressions either
(although they're just compiled to strings, which are).
You'll need these for paired arguments, same as Lissp.
These two expressions are normally equivalent in Hebigo.
```
print: 1 2 3 : :* 'abc'  sep "/"  # Hotword expression.
(print(1, 2, 3, *'abc', sep="/"))  # Bracketed expression.
```
But they would not be the same if `print` were a macro,
because macros can only be invoked in hotword expressions.

Control words may also be used as hotwords,
in which case they both begin and end with a colon.
This makes no sense at the top level (because strings are not callable),
but macros do use them to group arguments.

Unlike Lissp, normal string literals cannot contain literal newlines.
Use `\n` or triple quotes like Python instead.
(Recall that strings count as bracketed expressions.)

Hotword expressions may contain bracketed expressions,
but not the reverse, since bracketed expressions must be valid Python,
just like how the statements that the hotword expressions replace may contain expressions,
but Python expressions may not contain Python statements.

And finally, the `!` is an abbreviation for `hebi.basic.._macro_.`.
This obviously doesn't work in bracketed expressions either.
Hebigo has no other "reader macros".
