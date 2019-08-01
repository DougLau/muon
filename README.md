## MuON v0.2.0alpha

*Micro Object Notation*

**MuON** is a format for configuration files and data interchange — as
expressive as other formats, but [much simpler](WHY.md).

```
# Example MuON
sample: Text can contain "quotes" and colons (:)
the_dict:
    a: 13
    b: true
    poem: Once upon a midnight dreary, … 🌑 + 🌁?
    pi: 3.141592653589793
```

### Specification

**MuON** files are [UTF-8 encoded](https://en.wikipedia.org/wiki/UTF-8) with no
[byte-order mark](https://unicode.org/glossary/#byte_order_mark).  Since UTF-8
does not allow
[surrogate code points](https://unicode.org/glossary/#surrogate_code_point),
all characters are
[Unicode scalars](https://unicode.org/glossary/#unicode_scalar_value).

Each file is a *tree* of **dictionaries** (records), with an implied **root**.

Every line feed (U+000A) marks the end of a **line**.  There are three line
types: *blank*, *comment* and *definition*.  **Blank** lines contain no
characters.  **Comments** begin with zero or more spaces followed by a number
sign:

`# Example comment`

A **definition** creates a mapping between a **key** and **value**, like so:
```
key: value
```

With no **indents**, mappings are created for the root dictionary.  When a
mapping defines a new dictionary, subsequent definitions with an additional
indent are for that dictionary.

```
key in root: value in root
a:
    key in a: value in a
```

Every indent in a file must contain the same number of spaces, typically 2 to 4.
Only spaces (U+0020) are allowed, not tabs or other whitespace.  For nested
dictionaries, multiple indents are used.

```
mesa:
   # 3 space indent; ok
   comida: taco
   bandeja:
      # Two indents: 6 spaces
      nota: Lo dejo
```

A **key** is a sequence of one or more characters.  It must be `"`**quoted**`"`
if it contains a colon or begins with a space, quote mark or number sign.  When
*quoted*, all contained quote marks must be escaped by doubling:

```
# "skeleton" key must be quoted
"""skeleton""" key: value
```

Also, a key *should* be quoted if it contains any colon
[homoglyphs](https://en.wikipedia.org/wiki/Homoglyph#Unicode_homoglyphs),
[control characters](https://en.wikipedia.org/wiki/Unicode_control_characters)
or begins with [whitespace](https://en.wikipedia.org/wiki/Whitespace_character).

It is common to use keys directly within programming languages as
[identifiers](https://en.wikipedia.org/wiki/Identifier#In_computer_languages).
In this case, they should only contain ASCII alphanumeric and underscore
characters.

A **value** is a sequence of characters.  If empty, the space after the colon
in the definition is not required.

### Schema

A **schema** is a template with all values containing *types*.  It can either be
separate or prepended to a MuON file.  In either case, it begins and ends with a
line of three colons.

```
:::
# Sample schema
sample: text
the_dict: dict
    a: int
    b: bool
    poem: text
    d: float
:::
```

A **type** is one of the following:

Basic   | Optional |   List
------- | -------- | ---------
`text`  | `text?`  | `[text]`
`bool`  | `bool?`  | `[bool]`
`int`   | `int?`   | `[int]`
`float` | `float?` | `[float]`
`dict`  | `dict?`  | `[dict]`

**Optional** types (with a question mark) are not required — their definition
may not be present.

**Text** is a sequence of characters.  Because values cannot contain line
feeds, if one is needed a **text append separator** can be used on the next
line to *insert* one.  This is `:>` instead of the usual `: ` between the key
and value.  A line feed will be inserted between the text values.

When appending, a **blank** key can be used instead of repeating the key.
This is a sequence of spaces with the same length as the key.

```
text: value
    :>appended
```

**Bool** is a *boolean*: either `true` or `false`.

**Int** is an *integer* (whole number) in one of three forms:

  * *Decimal*: sequence of digits `0`-`9` (not starting with `0`).  May have a
    sign prefix `+` or `-`
  * *Binary*: `0b` followed by sequence of digits `0` or `1`
  * *Hexadecimal*: `0x` followed by sequence of digits `0`-`9`, `A`-`F` or
    `a`-`f`

An underscore may be inserted between digits to improve readability.

```
decimal_a: 4
binary_a: 0b1000
hexadecimal_a: 0x0F
decimal_b: +16
binary_b: 0b01_0111
hexadecimal_b: 0x2a
```

A **float** is a
[floating point](https://en.wikipedia.org/wiki/IEEE_754) number, made up of
these parts:
  1. Whole number part (same as decimal int)
  2. Fractional part (decimal point followed by sequence of digits `0`-`9`)
  3. Exponent part (`e` followed by decimal int)

One or both of the whole or fractional parts must be present, but the exponent
part is not required.  As with ints, underscores may be included.

The values `inf` and `NaN` stand for *infinity* and *not a number*,
respectively.  Either can be prefixed with a `+` or `-` sign.

```
float0: -1.5
float1: .0195
float2: 1e-10
float3: 13.835e12
float4: +inf
float5: 100
```

A **dict** is a *record* containing indented mappings from keys to values.

```
:::
things: dict
    alpha: int
    beta: text
:::
things:
    alpha: 15
    beta: What have you
```

Since dictionaries do not have *values* themselves, they can be used as a short
cut for the first contained mapping.  The only restriction is that mapping must
not have a dictionary type.

```
things: 15
    beta: alpha is equal to 15
```

A **list** is a type `[`enclosed in square brackets`]`.  Values are sequences
of the contained type, separated by spaces.  Like optional types, the
definition should be omitted if the list is empty.

```
:::
flags: [bool]
checks: [bool]
:::
flags: true false true true
# checks list is empty
```

Like text, lists can be **appended**.  All items are added to the end of the
list.

```
numbers: 2 4 6 8
       : 10 12 14 16
# same as numbers: 2 4 6 8 10 12 14 16
```

To append to a **list of dictionaries**, since the lines are not consecutive,
the key must not be blank.

```
:::
dict_list: [dict]
    a: int
:::
dict_list:
    a: 5
dict_list:
    a: 10
```

For a **list of text**, items are separated by spaces, just like other lists.
If spaces are needed, a **text value separator** `:=` can be used to treat the
entire value as a single text item.  The text append separator `:>` will also
behave the same way.

```
:::
text_list: [text]
:::
text_list: first second
         :=third item
         : fourth fifth sixth
         :>item
         : seventh
```
