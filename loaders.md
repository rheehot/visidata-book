# Main Doc about Loader infrastructure

- entire new standalone applications can be created with minimal code
- abstractions which do the heavy lifting for most data use cases
- the goal of this document is to provide a general overview of the big picture of how data is loaded and interfaced in VisiData.
    - to provide a context of understanding for other parts of the book
    - for more details on these various pieces, check out the rest of the book
- tutorials on building a loader will follow

# Loading data in VisiData at a glance

- rows.md contains reload() checklist for rows

- The main interface for VisiData is the Sheet.
- `Sheet` provides a stock async reload
- the data in sheets is loaded through calling `Sheet.reload()`
    - reload calls `self.iterload()`
        - self.iterload() provides an iterator for rows, by accessing the source file and yielding each rows 
        - reload uses this iterator to set the header based on the `skip` and `header` options
        - it then adds the rest of the rows (using `addRow`) to the sheet
        - if an ordering has been specified sorts the sheet
- `SequenceSheet` builds on this with a `setCols`, `newRow` and `addRow` that work best with formats whose row objects are an iterator of some sort (e.g. list, tuple)

- in the tutorial, we will cover x cases
    - creating an iterload, defining static Column objects

# How to create a loader for VisiData

- with tutorials, start at the lowest level, and then work your way up
    - the higher levels come with stock tooling for interfacing with the most common aspects of data formats

The process of designing a loader is:

1a. create a **Sheet** subclass;
1b. tell `vd.filetype()` about the extension and new **_Sheet**;
2. write an `iterload()` function to yield each **row**;
3. define the available **columns**;
4. define sheet-specific **commands**/**options** to interact with the rows, columns, and cells.

## 1. Create a Sheet subclass

When VisiData tries to open a source with filetype `foo`, it searches for the filetype in the dict `vd.filetypes`. If it is present, `vd.filetypes` should return an instance of `Sheet`, or raise an error.

    class FooSheet(Sheet):
        rowtype = 'foobits'

    vd.filetype('foo', FooSheet)

- The `rowtype` is for display purposes only.  It should be **plural**.
When a sheet is loaded in VisiData, the number of rows, along with the rowtype, will be displayed on the bottom right status.
- `vd.filetype('foo', FooSheet)` stores the key 'foo', along with `FooSheet` in `vd.filetypes`


## 2. Load data into rows, and yield them one-by-one

`reload()` is called when the Sheet is first pushed, and thereafter by the user with `^R`.
The stock `reload()` iterates through the rows returned by `iterload`.

The stock `reload()` suits most cases just fine.
Each loader then defines an `iterload`, which uses the Sheet `source` to populate and then yield `rows`:

    class FooSheet(Sheet):
        rowtype = 'foobits'  # rowdef: foolib.Bar object
        def iterload(self):
            for r in crack_foo(self.source):
                yield r

- A `rowdef` comment should declare the **internal structure of each row**.

### Supporting asynchronous loaders

Large enough datasets will cause the interface to freeze.
Fortunately, the stock `reload` and the `iterload` structure results in an [async](/docs/async) loader on default.
Since rows are yielded **one at a time**, they become available as they are loaded, and `reload` itself is decorated with an `@asyncthread`, which causes it to be launched in a new thread.

Further things to take into account:
- All row iterators should be wrapped with [`Progress`](/docs/async#Progress).  This updates the **progress percentage** as it passes each element through.
- Do not depend on the order of `rows` after they are added; e.g. do not reference `rows[-1]`.  The order of rows may change during an asynchronous loader.
- Catch any `Exception`s that might be raised while handling a specific row, and add them as the row instead.  If `Exception` handling is missing within iterload, rows will stop loading upon hitting an `Exception`. Never use a bare `except:` clause or the loader thread will not be cancelable with `Ctrl+C`.

#### Progress and Exception example
    class FooSheet(Sheet):
        ...
        def iterload(self):
            for bar in Progress(foolib.iterfoo(self.source.open_text())):
                try:
                    r = foolib.parse(bar)
                except Exception as e:
                    r = e
                yield r

Test the loader with a large dataset to make sure that:

- the first rows appear immediately;
- the progress percentage is being updated;
- the loader can be cancelled (with `Ctrl+C`).

## 3. Enumerate the columns

Each sheet has a unique list of `columns`. Each `Column` provides a different view into the row.

    class FooSheet(Sheet):
        rowtype = 'foobits'  # rowdef: foolib.Bar object

        columns = [
            ColumnAttr('name'),  # foolib.Bar.name
            Column('bar', getter=lambda col,row: row.inside[2],
                          setter=lambda col,row,val: row.set_bar(val)),
            Column('baz', type=int, getter=lambda col,row: row.inside[1]*100)
        ]

In general, set `columns` as a class member.  If the columns aren't known until the data is being loaded,
**Sheet**'s `__init__()` will check if `self.columns` was set and add each column at the earliest opportunity.

### Column properties

Columns have a few properties, all of which are optional arguments to the constructor except for `name`:

* **`name`**: should be a valid Python identifier and unique among the column names on the sheet. (Otherwise the column cannot be used in an expression.)

* **`type`**: can be `str`, `int`, `float`, `date`, or `currency`.  By default it is `anytype`, which passes the original value through unmodified.

* **`width`**: the initial width for the column. `0` means hidden; `None` (default) means calculate on first draw.

* **`fmtstr`**: [strftime-format](https://docs.python.org/3/library/datetime.html#strftime-strptime-behavior) for `date`, or a [new-style Python format string](https://docs.python.org/3/library/string.html#formatstrings) for all other types.

* **`getter(col, row)`** and/or **`setter(col, row, value)`**: functions that get or set the value for a given row.

#### The `getter` lambda

The getter is the essential functionality of a `Column`.

In general, a `Column` constructor is passed a `getter` lambda.
Columns with more complex functions should be subclasses and override `Column.calcValue` instead.

The `getter` is passed the column instance `col` and the `row`, and returns the value of the cell.
If the sheet itself is needed, it is available as `col.sheet`.

The default getter returns the entire row.

#### The `setter` lambda

The `Column` may also be given a `setter` lambda, which allows the in-memory row to be modified.
The `setter` lambda is passed the column instance `col`, the `row`, and the new `value` to be set.

In a Column subclass, `Column.setValue(self, row, value)` may be overridden instead.

By default there is no `setter`, which makes the column read-only.

### Builtin Column Helpers

There are several helpers for constructing `Column` objects:

* `ColumnAttr(colname, attrname, **kwargs)` gets/sets the `attrname` attribute from the row object using `getattr`/`setattr` (as in `row.attr`).  The `attrname` defaults to the `colname` itself.
  `ColumnAttr` is useful when the rows are Python objects.

* `ColumnItem(colname, itemkey, **kwargs)` uses the builtin `getitem` and `setitem` on the row (as in `row[itemkey]`).  The `itemkey` also defaults to the `colname` itself.
  This is useful when the rows are Python mappings or sequences, like dicts or lists.

* `SubColumnItem(subrowidx, origcol, **kwargs)` proxies for another Column, in which its row is nested in another sequence or mapping.
This is useful on a sheet with augmented rows, like `tuple(orig_index, orig_row)`; each column on the original sheet would be wrapped in a `SubColumnItem(1, col)`, since `orig_row` is now `row[1]`.  Used in joined sheets.

* `SubColumnAttr(attrname, origcol, **kwargs)` does the same, but with an attribute on the row instead.

Recipes for a couple of recurring patterns:

- columns from a list of names: `[ColumnItem(name, i) for i, name in enumerate(colnames)]`
- columns from the first sample row, when rows are dicts: `[ColumnItem(k) for k in self.rows[0]]`

## 4. Define Sheet-specific Commands

`<Sheet>.addCommand()` and `globalCommand()` have the same signature:

`[...]Command(default_keybinding, longname, execstr)`

    class FooSheet(Sheet):
        ...

    FooSheet.addCommand('b', 'reset-bar', 'cursorRow.set_bar(0)')

- Optionally, a reasonably intuitive and mnemonic default keybinding could be chosen.
- The longname is a mandatory argument and allows the command to be rebound by a more descriptive name, and for the command to be redefined for other contexts (so all keystrokes bound to that command will take on the new action).

See the [commands design document]() and [commands checklist]() for more details.

## Full Example

This would be a completely functional read-only viewer for the fictional foolib.  For a more realistic example, see the [annotated viewtsv](/docs/viewtsv) or any of the [included loaders](https://github.com/saulpw/visidata/tree/stable/visidata/loaders).

    from visidata import *

    class FooSheet(Sheet):
        rowtype = 'foobits'  # rowdef: foolib.Bar object
        columns = [
            ColumnAttr('name'),  # foolib.Bar.name
            Column('bar', getter=lambda col,row: row.inside[2],
                          setter=lambda col,row,val: row.set_bar(val)),
            Column('baz', type=int, getter=lambda col,row: row.inside[1]*100)
        ]

        def iterload(self):
            import foolib

            for bar in Progress(foolib.iterfoo(self.source.open_text())):
                try:
                    r = foolib.parse(bar)
                except Exception as e:
                    r = e
                yield r

    FooSheet.addCommand('b', 'reset-bar', 'cursorRow.set_bar(0)')

    vd.filetype('foo', FooSheet)

## Extra Credit: create a saver

A full-duplex loader requires a **saver**.  The saver iterates over all `rows` and `visibleCols`, calling `getValue`, `getDisplayValue` or `getTypedValue`, and saves the results in the format of that filetype. Savers can be decorated with `@Sheet.api` in order to make them available to all sheet types.

    @Sheet.api
    def save_foo(path, sheet):
        with path.open_text(mode='w') as fp:
            for i, row in enumerate(Progress(sheet.rows)):
                for col in sheet.visibleCols:
                    foolib.write(fp, i, col.name, col.getValue(row))

The saver should preserve the column names and translate their types into foolib semantics, but other attributes on the Columns should generally not be saved.

---

# Building a loader for a URL schemetype

When VisiData tries to open a URL with schemetype of `foo` (i.e. starting with `foo://`), it calls `openurl_foo(urlpath, filetype)`.  `urlpath` is a `UrlPath` object, with attributes for each of the elements of the parsed URL.

`openurl_foo` should return a Sheet or call `error()`.
If the URL indicates a particular type of Sheet (like `magnet://`), then it should construct that Sheet itself.
If the URL is just a means to get to another filetype, then it can call `openSource` with a Path-like object that knows how to fetch the URL:

    def openurl_foo(p, filetype=None):
        return openSource(FooPath(p.url), filetype=filetype)

---

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
