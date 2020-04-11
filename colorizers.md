# Colorizers

## Examples

### Colorizer changes cells to 'color_error' when value is negative

    rc = RowColorizer(1, 'color_error', lambda s,c,r,v: v < 0)
    sheet.addColorizer(rc)

- This colorizer takes a precedence level of 1 (higher number are higher precedence).
- If the lambda expression is True, `options.color_error` is applied to the row.
- The lambda parameters are `sheet, column, row, value`.

### Add shortcut and undoable command to reverse color of cells with value 3

```python
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


## Overview

VisiData's various drawing tools (`clipdraw`, `plotlines`, etc) color using [curses attributes](https://docs.python.org/3/howto/curses.html#attributes-and-color), integers where each bit communicates how text should be displayed on the terminal.

While, curses attributes are versatile and efficient for the internal functions, they are not really human readable.

An API was therefore built with the aims:
1) To enable users and developers to easily tailor VisiData's coloring to their liking;
2) For ease of coloring of rows, columns, cells, and notes based on their properties;

To explore how coloring works in VisiData, we are going to cover code in sheets.py and color.py. sheets.py py contains the bulk of VisiData's drawing code, and color.py contains the infrastructure specific to coloring.

#### `theme(name, default, helpstr)`

The first step in the coloring workflow is to create a theme.

Themes are color options. Arguments are the `name` (must begin with `color_`), `default` (contains the color string), and `helpstr` (a helpful description). e.g. `theme('color_graph_hidden', '238 blue', 'color of legend for hidden attribute')`

Color strings contain a list of space-delimeated words that represent colors. Upon startup, VisiData's `ColorMaker` will try to compute these color strings by parsing each part as a [color](https://github.com/saulpw/visidata/blob/develop/visidata/color.py#L72) (red green yellow blue magenta cyan white, or a number if 256 colors available) or [attribute](https://github.com/saulpw/visidata/blob/develop/visidata/color.py#L76) (normal blink bold dim reverse standout underline). A 256-color palette is not guaranteed, however, and in that case a color like `238` would not exist.

With default colors, it is recommended that the second term be a basic color, to be used as a fallback. If the `238` is found, then it's used preferentially. (Attributes however are always applied).

Upon startup, a `ColorMaker` object is created. For each `theme()`, it translates the color strings into curses attributes. The setup `ColorMaker` is then importable in VisiData as `colors`. It contains a key for each color option name associated with its converted curses attribute (e.g. colors.color_graph_hidden or colors['color_graph_hidden']).


#### `RowColorizer`, `CellColorizer`, `ColumnColorizer(precdence, coloropt, func(sheet, col, row, value))`

A colorizer object is a simple named tuple, which VisiData employs to do property based coloring. Colorizers come in the form of `RowColorizer` (colors entire rows), `CellColorizer` (colors a single cell), and `ColumnColorizer` (colors an entire column).

Each of the named tuples contain the keys for 'precedence',  'coloropt', and 'func'.

Curses can combine multiple non-color attributes, but can only display a single color. Precedence is a number which indicates the priority a color has (e.g. if a row is selected, we want the selection color to trump the default color). A higher precedence color overrides a lower. `coloropt` is the color option name (like 'color_graph_hidden')

`func(sheet, col, row, value)` is a lambda function which should return a True value for the properties when coloropt should be applied. If coloropt is None, func() should return a coloropt (or None) instead.

#### `ColorAttr`

`ColorAttr` is a named tuple used to help with dynamic coloring, while keeping precedence in mind. It contains a color, attributes list, and a precedence. It stores the attributes and color separately so that the color can be updated while keeping all of the attributes. ColorAttr().attr returns the curses attribute for the color combined with all of the stored attributed.

- `colors.get_color(coloropt)`
    - public api
    - should be used instead of creating a new ColorAttr directly
    - returns the `ColorAttr` for the `theme()` with the name `coloropt`

- `update_attr()`
    - internal use only
    - takes an existing ColorAttr and updates it with another one's color, taking precedence into account
    - if the precedence is lower, no change happens to the color
    - attributes always stack


- `_getColorizers()`
    - internal use only
    - returns all colorizers in precedence order; highest precedence first

- `_colorize()`
    - internal use only
    - returns a ColorAttr for the given col/row/value
        - the curses attribute will have a list of color option names sorted highest-precedence color first
    - `_colorize()` calls all of them to see if they fire'


#### ColorsSheet

Pressing space followed by the longname `colors` opens up the `ColorsSheet`. The `ColorsSheet` contains every available color along with its curses RGB code.


## misc style notes
- selection and current row should trump other color options
- current row must be solid reverse (no breaks for seps)
