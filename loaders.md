
### Passthrough options

Loaders which use a Python library (internal or external) are encouraged to pass all options to it through the `options("foo_")`.  For modules like csv which expose them as kwargs to some function or constructor, this is very easy:

    rdr = csv.reader(fp, **csvoptions())

## specific loaders

### csv

The `csv_` options themselves are from https://docs.python.org/3/library/csv.html#csv-fmt-params

# [dev] Loading process

It's important that sheets can be pushed without being loaded.
If many sheets are pushed at once (whether from the cmdline, or with `g+Enter` on an index sheet), it would degrade performance to load them all immediately in parallel.
Only one sheet can be seen at a time anyway.

[In 1.x, Sheets were loaded the first time they are pushed.  This was not a perfect solution.]

In 2.x, sheets are loaded (by calling `Sheet.reload()`) if its `.rows` is the `UNLOADED` object before `draw()` is called.

## IndexSheet

If a format contains multiple sheets, the toplevel loader should inherit from `IndexSheet`, which has a rowdef of `Sheet`.

Thus `open-row` (`Enter`) pushes the sheet onto the stack.
