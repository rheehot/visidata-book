# Frequency Table (Shift+F)

The frequency table is one of VisiData's most popular and powerful features. It's like 'facet' exploration in other tools.  It wasn't in the tpmenu tool I built at F5, but it was there in concept as part of the initial design doc, before any code had been written.[2]

A frequency table assorts rows from its source sheet into bins, according to their values as computed by a set of "group by columns"[1] on the source.
Proxies to these group-by columns are the key columns on the frequency table.  [`reloadKeyCols(groupByCols)`]

## Sheet commands

- `freq-col`/`Shift+F` cursor column
- `freq-keys`/`g Shift+F` visible key columns
- `freq-summary`/`z Shift+F` summarize selected rows

### on Frequency Table itself

- `dive-row`/`Enter`
- selection `s/t/u`
- selecting or unselecting rows on the frequency table, cascades that selection/unselection back to the source sheet.  This is useful for selecting individual values manually, quitting back to the source sheet, and then dup'ing it.
  - this should apply to most if not all selection actions, but notably "undo" will not undo the source selections.

## binning

The assorting is done according to the *formatted* *typed* values in the key columns.
Thus changing either the `type` or the `fmtstr` of the column may change the binning substantially.
"What you see is how it's binned".

### numeric binning

Numeric columns (int, float, date, currency) will be binned in automatically-determined ranges if `options.numeric_binning` is set.

Only one numeric column allowed in the group key columns.

Bins are evenly distributed over the numeric range.

#### number of bins

- sqrt(nRows)
- `options.histogram_bins`
- [degenerate binning] if more bins than int vals, just use the values themselves.

### error bins

error bins are created according to their display value.  In general computational errors

## aggregated columns

## commands


## options

- `disp_histogram`
- `disp_histolen`

# Pivot Table

a bit more just like the frequency table

## commands

- `Enter` dives into the row
- `z Enter` dives into the cell (the rows used to calculate up the single cell value)

[2]Originally, it was just the value and the count, and @helenvcook immediately asked for the percentage.
[1] so named because of SQL's `GROUP BY` clause
