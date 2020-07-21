## Overriding delete

When overriding any of the row deleting mechanisms, be sure to loop over a *copy* of the rows, not the rows themselves. The list of rows might change over time.
