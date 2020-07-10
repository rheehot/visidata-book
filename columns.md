`SettableColumn`:
    - https://github.com/saulpw/visidata/issues/413
    - SettableColumn's putValue adds rows and columns to sheet.cache, instead of sheet.rows. copyRows grabs the cliprows from sheet.rows. as a result of this, if you try to copy a SettableColumn's cells, you will get blank values.

## Column.recalc()

Does these things:

1. Fixes the column name, if applicable (stripping spaces, and forcing a valid column name if `options.force_valid_colnames` is True.

2. Clears its cache, if it has one

3. Associates the column with the Sheet that it is a part of (as the .sheet member).

[Column.recalc()]() must be called for every column, primarily to set its .sheet to the sheet it's on.
Sheet.addColumn does this automatically, and this is the preferred way to add dynamic columns .
You can also pass `columns` as a kwarg to a Sheet constructor as a list of Columns, or at the class level with a list of static Columns.

If for some reason you need to have dynamic columns without addColumn, you must call Sheet.recalc() (which calls Column.recalc() on every column).


## Column.cache

To set a column cache:

pass cache=True to constructor.
