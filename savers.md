# [dev] api

- `Sheet.itervalues(*cols, format=False, trdict={})`
  - use this for performance and error handling!  if you just use getDisplayValue then it's doing a lot of extra work each time, which can really add up for large datasets.

