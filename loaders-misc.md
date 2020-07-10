
Lazy, asynchronous loading is baked into the design of VisiData.

# options.skip and options.header

These options apply to a handful of formats:

  - tsv/csv and variants
  - fixed rows
  - xls (and other free-form spreadsheets)
  - possibly html

options.skip=N will skip N lines before the header, without parsing.

options.header=N will consume the next N rows to deduce the columns, and use the values from those rows as the column names.

Loaders which subclass from SequenceSheet will have these options handled for them.
