## Column Types

Every column has a `type`, which affects how it is parsed, displayed, grouped, sorted, and more.
The classic VisiData column types are:


### Commands to set the column type

|type    |description    |numeric  |command    |keystrokes    |mnemonic                             |
|--------|---------------|---------|-----------|--------------|-------------------------------------|
|str     |string         |         |type\-str  |~             |looks like a little piece of string  |
|date    |date/time in any decipherable format|Y        |type\-date |@             |"at" sign                            |
|int     |integer        |Y        |type\-int  |\#            |"number" sign                        |
|float   |decimal        |Y        |type\-float|%             |"percent" sign                       |
|currency|decimal with leading or trailing characters, and parsed according to locale|Y        |type\-currency|$             |"dollar" sign                        |
|vlen    |size of a container, or length of a string|Y        |type\-vlen |z\#           |much like an "integer", but specifically for size|
|anytype |default generic type|         |type\-anytype|z~            |even more generic than a string      |

Note that all types except `anytype` and `str` are considered numeric.

{mnemonic}
The keybindings for settings types are in a row on the shifted top left keys on a US QWERTY keyboard.


### Nulls

In version 1.3 and earlier, there were several options like `empty_is_null` to determine which values counted as null.  But this was poorly-defined and not implemented consistently.  After 1.3 it was simplified to:

1. The null values are:
   a. None
   b. the options.null_value (default None)

2. Null values interact with:
   a. aggregators: the denominator counts only non-null values
   b. the Describe Sheet: only null values count in the nulls column
   c. the `fill-nulls` command
   d. the `melt' command only keeps non-null values
   e. the `prev-null` and `next-null` commands (`z<` and `z>`)

#### `options.null_value`

This option can be used to specify a null value in addition to Python `None`.  This value is typed, so can be set to `''` (empty string) or another string (like `NA`), `0` or `0.0`.


#### [dev] api

- `isNullFunc()` return a function that takes a value and returns whether or not it is null.
There is no direct isNull function, because the possible null values can change at runtime, and getting an option value is too expensive to do in a bulk operation.

##### Properties of visidata type classes

where T is the type class (like `int`, `date`, etc) and `v` is an instance of that type.

- callable
    - T(): (no value): a reasonable default value of this type
    - T(v): exact copy of value
    - T(str): converts from a reasonable string representation
-  `__name__`: shown in ColumnsSheet.type

##### Properties of objects returned by T()

- comparable (for sorting)
- hashable
- roundable (for binning)
- formattable

##### `defType()` registers a type in the typemap

- .typetype: actual type class T above
- .icon: unicode character in column header
- .fmtstr: format string to use if fmtstr not given
- .formatter(fmtstr, typedvalue): formatting function (by default `locale.format_string` if fmtstr given, else `str`)

getType(T) -> vdtype


### [dev] rationale for `vlen` type

VisiData types must be both callable and idempotent.

The essential problem with Python builtin `len` as type is that it isn't idempotent; len(len(L)) is not valid--all other VisiData types can do that.

There's some special case code for anytype detecting that the cell is a list or dict, and displaying it differently.
Of course I wanted to sort by the length, so I would wind up creating a `len(col)` and setting it to int and then sorting by that.

But at one point, I thought, could the base value be a list, and len be the "type"?  I tried it out, and it basically worked.
But, the essential problem with len as type is that it isn't idempotent; len(len(L)) is not valid--all other VisiData types can do that.

So now there is a `vlen` type, which subclasses `int` and provides a `__len__`.  Useful for columns with compound objects; `z#` sets a column to `len` type.
