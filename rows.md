- `newRow`
    - adds a new row obj positioned below the current cursor
    - Sheet class on default calls `type(sheet)._rowtype()`
- `._rowtype`
    - creates a new acceptable row object for its bound sheet
    - on default creates a dictionary
- `rowtype`
    - str which is the plural of the English for the type of row
    - used in the bottom right status (e.g. '5 bins' where 'bins' is the rowtype)
- `rowdef`
    - comment which describes the row object; development ai

## Misc
When `defermods.py` is installed, new rows on loaded sheets are not coloured. On new sheets, they are coloured green.
This is because new rows are added to deferredAdds at the level of the Sheet class. Loaded sheets tend to have their own `newRow` which overrides this.
