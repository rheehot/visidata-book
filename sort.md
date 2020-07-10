# sort.py

Sort functionality is not an intrinsic operation in core VisiData.
But it can be added with 50 lines of code

## Sheet property: `ordering`

The 'sort' module allows the user to set an intended 'ordering' of rows on the sheet.

This ordering

## Sheet commands: `sort-*`

- Command|sort-asc|...


## API

- `sheet.orderBy(*cols, reverse=False)`
  - cols are each added to the existing ordering (all in reverse if reverse=True)
  - if cols[0] is None, the ordering is reset before adding the rest of the cols
- `sheet.sort()`
  - applies existing ordering
  - should be async, sets at end
  - override to provide custom sorting behavior
    - but use ordering to determine how to sort
  - provide Progress in sortkey
- `sheet.reload()` should also obey the ordering
