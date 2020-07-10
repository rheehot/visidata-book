## Options

### Options hierarchy

1. runtime override (e.g. via OptionsSheet)
2. jl
"global": default specified in option() definition
        2. "override": in order of program execution:
            a. .visidatarc
            b. command-line options, applied on top of the overrides in .visidatarc)
            c. at runtime via 'O'ptions meta-sheet
        3. objname(type(obj)): current sheet class and parents, recursively
        4. objname(obj): the specific sheet instance
            a. can override at runtime, replace value for sheet instance


### `options-sheet` (`Shift+O`) for this specific sheet options

### `options-global` (`g Shift+O`) for global options
  - changes affect all sheets unless overridden for a specific sheet

###

To override the value of an option, use Python syntax in `.visidatarc`:

    options.clipboard_copy_cmd = 'xclip -selection primary'
    options.min_memory_mb = 100
    options.quitguard = True

Option names should use the underscore for word separators.  On the command-line, underscores are converted to dashes:

    $ vd --min-memory-mb=100

Option names should have 20 characters or fewer.


### [dev] api

To get the value of an option:

    - `foo = options.num_burgers`
    - `options['num_burgers']`
    - `options.get('num_burgers', obj)`
        - in the context of obj (sheet, SheetClass, Sheet, 'global', 'override'; extensions should use sheet or SheetClass)

    - `options.num_burgers = 40`
    - `options['num_burgers'] = 40`
        - useful for option pass-throughs

    - `options.get('num_burgers', obj)`
        - in the context of obj (sheet, SheetClass, Sheet, 'global', 'override'; extensions should use sheet or SheetClass)

Getting the current value of an option is an expensive operation.  Do not use in inner loops.

To declare an option:

    option('num_burgers', 42, 'number of burgers to use', replay=True)

To get a dict of all options starting with `foo\_` (useful for loader options):

    options('foo_')


- command-line options need to override class-specific options
- but some classes need to forcibly set what value to use, and  for e.g. 'header' (a DirSheet should not consume the first few files to construct the column names; or at least, it should be quite difficult)
- ultimate solution is probably sheet specific options on the cmdline
- objects could set for now, can 




### ref

- OptionsObject.get(k, obj)
- OptionsObject.set(k, v, obj)
- OptionsObject.unset(k, obj)
  - different than setting v=None
- OptionsObject.getdefault(k)
   - use for options metasheets
- OptionsObject.setdefault(k)
   - options.__call__()

- option(optname, default, helpstr, replay=True)
   - optname
        - Use '\_' for a word separator.
        - all option names are in a global namespace
   - default
        - The type of the default is respected, with strings and other types being converted, and an `Exception` raised if conversion fails.  A default value of None allows any type.
   - helpstr
        - in ^H (manpage) and g^H (command list)
   - `replay` (kwarg, bool, default False) indicates if changes to the option should be stored in the command log.
        - If the option affects loading, transforming, or saving, then it replay should be True.

`theme()` should be used instead of `option()` if the option has no effect on the operation of the program, and can be overrided without affecting existing scripts.  The interfaces are identical.
Theme options should start with `disp_` and `color_` and cannot be passed on the command-line.

#### lesser used

   - BaseSheet.optionsSheet
   - vd.globalOptionsShet

