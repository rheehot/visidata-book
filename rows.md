## Rows

The primary element of each VisiData Sheet is its list of **rows**.

In VisiData, a row can be any type of object, though every row on a Sheet should be the same type of object, called the `rowtype`.
The number of rows and the rowtype of the active sheet are always shown in the lower right status bar.

   `1234 rows`

The rowtype for source sheets defaults to `rows`:

The displayed rowtype is a reasonable indicator of which sheets are row-compatible.
For instance, an .html document opens up as a list of the available sheets contained within it, and the rowtype is shown as `sheets`.
The Sheets Sheet (`S`) also shows a rowtype of `sheets`.
This means that you can copy rows from the HTML index, and paste them into the Sheets Sheet.

### [dev] rowdef

In the code, each Sheet[1] subclass has these members:

- `rowtype`: the displayed rowtype (in plural form)
    - `# rowdef`: comment next to the rowtype indicating the Python structure of each row
- `rows`: a sequence of rows
- `__len__`: all sheets have a length; some measure of the number of elements in the dataset
- `nRows`: the number of rows, same as len(sheet)
- `addRow(r, idx=None)`: insert a row into the list of rows at idx (default idx=None means append at end).
- `_rowtype`: callable; set to create a new empty row (default `object`)

[1] even non-tabular Sheets have a fundamental list of elements.  These are called 'rows' internally ("objects" visibly) even though they may not be strictly row-like in the tabular sense.

### Row requirements

- must be unique objects: id(row) must be unique or e.g. selection won't work


### [dev] Sheet loading

#### rows = UNLOADED

Before first `reload()`, `rows` is set to the UNLOADED sentinel object.
Then, when the sheet is first pushed, the reload is triggered.

This is so many hundreds or thousands of proto-sheets can be created, representing the data that will be loaded when and only when the user is interested.
But, it's not just an empty list, so empty sheets won't be reloaded over and over again.

#### `reload() checklist for rows`

- set the rowtype at the top of the class
- rowdef comment closeby
- reload() should be async for any non-constant-time loader
- create new .rows before loading (`self.rows = []`)
   - do not use .clear(); it may be referenced elsewhere
- call addRow(row) for each row

## Misc
When `defermods.py` is installed, new rows on loaded sheets are not coloured. On new sheets, they are coloured green.
This is because new rows are added to deferredAdds at the level of the Sheet class. Loaded sheets tend to have their own `newRow` which overrides this.
