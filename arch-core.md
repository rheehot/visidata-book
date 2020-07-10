#

## Minimum Viable VisiData

The essentials are:

- tabular rendering: TableSheet.draw()
- command dispatch from input
  - being able to specify commands as code fragments aided early development
  - exec()/eval() function, whether for the target language or a different one, is handy for configurability, as well as more advanced functionality like ExprColumn
- rows stored as objects, columns as functions

### recommendations

  - object-oriented
    - 
