## Join

VisiData provides several processes for combining the columns and/or rows of two or more sheets. These are all `join-` commands (bound to `&` on default).

Most joins pass a list of sheets to the **JoinSheet** constructor. Sheets are then joined based on common values present within their key columns.

For example, to do the equivalent of this pandas command:

```
df_cd = pd.merge(df_1, df_2, how='inner', on = 'Id')
```

in VisiData:

1. load the two sheets df_1 and df_2.
2. set `Id` to be the key column on both sheets
3. press `&` (`join-top2`) and type `inner` (jointype).

## JoinSheet
#### rowdef
The core of the **JoinSheet** is the `combinedRow`. Each `combinedRow` contains references to one source row from each sheet where the keys match.  If a source sheet does not have a row with a particular key, the corresponding row reference will be a `None`.

If you push the [python object]() (`^Y`) for a joined row, you will see a list of internal row objects from those other sheets.

Since the joined row refers back to the joined sheet's sources, if you modify the joined row, it modifies the source row.
Conversely, if the source is modified, the joined row may become obsolete.

In a fashion, JoinSheets are similar to an [SQL view](https://en.wikipedia.org/wiki/View_(SQL)). They are connected to their origins.

The selected jointype determines which of the `combinedRow`s are retained in the final **JoinSheet**. Based on the selected jointype, some of the source sheet rows may Be `None` instead of a row.

- `inner`
    - keep **only rows which match keys on all sheets**
    - none of the source sheet rows can be `None`
- `outer`
    - refers to 'left outer' join from SQL. to do a 'right outer' join, reverse order of the sheets in the **SheetsSheet**
    - keep **all rows from first joined sheet**
    - the **second** (or more) source sheet rows might be `None`
    - the first source sheet rows cannot be None
- `full`
    - full cross join
    - keep **all rows from all sheets** (union)
    - either might be `None`
- `diff`
    - keep **only rows with keys NOT in all sheets**
    - one of the source sheet rows MUST be `None`
- `append`
    - keep all rows from all sheets (concatenation)
- `extend`
    - keep **sheet type and all rows from first sheet**, and extend with columns from other sheets
    - extend is a special form of `outer` for VisiData
        - with most other jointypes, a view is made into the sheets' data
        - Some sheets are special, and represent more than just data (e.g. the DirSheet representing files). They have actions that operate specially on those rows.



### helpers
#### commands
`&` was chosen as the default keybinding, due to its association with ANDing.
`join-sheets` merges the selected sheets (on the **SheetsSheet**) with visible columns from all, keeping rows according to jointype.
`join-sheets-top2` merges the top two sheets in the sheets-stack.
`join-sheets-all` joins all of the sheets in the sheets-stack.

#### createJoinedSheet
Takes a list of sheets and a jointype, and calls the relevant Sheet constructor.
When the jointype is 'append', a **ConcatSheet** is initiated. When it is 'extend', `ExtendedSheet_reload` is run. For every other jointype, a **JoinSheet** is built.

#### groupRowsByKey

A crucial helper which efficiently constructs the `combinedRow` by looking for key matches between sheets.

#### joinkey

Returns the value of the key we are joining on, for a particular sheet and row.
Important to note that it uses the `getDisplayValue` instead of the `getValue`. This is so that any formatting specified in the column affects the join.

#### JoinKeyColumn - calcValue/putValue
`calcValue` for keys checks for consistency and if not present prompts for a reload. An inconsistency occurs when a key value has been changed on a single source sheet.
If a key column value has been changed on a `JoinSheet`, `putValue` fans out the change to all of the source sheets.

#### ConcatSheet
The new sheet contains a concatenation of all of the rows from all source sheets. Unlike other join types, no key column is required.

#### ExtendedSheet_reload
A function, which is passed a copy of the base sheet. Upon this copy, it builds the join. This is so that the new merged sheet has the exact sheet type as the base.
