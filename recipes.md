## Colorizer Examples

### Colorizer changes cells to 'color_error' when value is negative

    rc = RowColorizer(1, 'color_error', lambda s,c,r,v: v < 0)
    sheet.addColorizer(rc)

- This colorizer takes a precedence level of 1 (higher number are higher precedence).
- If the lambda expression is True, `options.color_error` is applied to the row.
- The lambda parameters are `sheet, column, row, value`.

### Add shortcut and undoable command to reverse color of cells with value 3

```
@Sheet.command('1', 'equal-3', 'reverse cells == 3')
def equal3(sheet):
    c = CellColorizer(0, 'reverse', lambda s,c,r,v: v == 3)
    sheet.addColorizer(c)
    vd.addUndo(sheet.removeColorizer, c)
```
- The @Sheet.command decorator adds a command called 'equal-3', and keyboard command '1'
- This adds a colorizer with precendence level of 0 (higher numbers override colorizers with lower precdence).
- If the cell value is 3, then the cell will be 'reversed' color
- This adds an Undo command using `sheet.removeColorizer`, to remove the colorizer.

### Alternating Row Colorizer

    sheet.addColorizer(RowColorizer(10, "color_row_alt", lambda s,c,r,v: r and s.rowindex(r) % 2))'

- This requires the [`rownum` plugin](https://github.com/saulpw/visidata/blob/develop/plugins/rownum.py)
    - This plugin provides the rowindex() method, which provides an integer index of the row.
    - It also provides a prev() method, which can be used to inspect the previous row.
-  Alternate column colors could be done using Sheet.visibleCols.index(c)
-  The `r` parameter to the lambda can be None, like when called on the column headings.
    - Use the leading `r and expression` idiom to protect against None values.
- This expects there to be `color_row_alt` option, which needs to be added in .visidatarc.
    - A color specification could be used instead like 'blue', 'reverse' or '238'

### Default Colorizers

Below are the default Colorizers from the source code.


    # This colorizer is used to color the column header cells.  If the row r
    # is None, the current cell is a header cell.
    CellColorizer(2, 'color_default_hdr', lambda s,c,r,v: r is None)

    # This colorizer is used to color the current column the cursor is in.
    # If the column c is the same as s.cursorCol, the cell being colored is
    # in the same column as the cursor.,
    ColumnColorizer(2, 'color_current_col', lambda s,c,r,v: c is s.cursorCol)

    # This colorizer is used to color the key columns.  If the column c is a key column
    # if c.keycol is True.
    ColumnColorizer(1, 'color_key_col', lambda s,c,r,v: c and c.keycol)

    # This colorizer always triggers since it returns True.  It has a precedence of 0,
    # which is lower than all the other builtin colorizers.  So this colorizer's color
    # is applied only if none of the other colorizers return True.
    CellColorizer(0, 'color_default', lambda s,c,r,v: True)

    # This colorizer is used to set the color for selected rows. It uses the sheet.isSelected
    # method on the the row r, to determine if the row is selected.
    RowColorizer(2, 'color_selected_row', lambda s,c,r,v: s.isSelected(r)),

    # This colorizer is used to set the color on error rows.  Only if the row r is an exception does this rule trigger.
    RowColorizer(1, 'color_error', lambda s,c,r,v: isinstance(r, (Exception, TypedExceptionWrapper))),

