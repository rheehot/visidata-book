## Settings

[in `visidata/settings.py`]

Options, Commands, and Key Bindings are all settings that are managed in a similar way.
They can each effectively be set, in order of lowest precedence to highest:

- globally
- for all types of tabular Sheets,
- for a specific type of derived Sheet,
- from the command-line
- at runtime via sheet-options for a specific sheet

### .visidatarc

Options and other settings can be set in .visidatarc as simple Python:

   `options.min_memory_mb = 50`

All Python code in .visidatarc is executed and made available to column expressions and other components.

### `open-config` (`g Shift+O`) opens the .visidatarc file for editing

{see commands.md}

### [dev] settings api

- loadConfigFile(fnrc)

These are all valid contexts:

a. `options.foo`: get in context of current sheet; set for runtime override ('override')
b. `vd.options.foo`: get in context of current sheet; set as global default ('global')
c. `sheet.options.foo`: get and set in context of given sheet
d. `Sheet.options.foo`: plugins without Sheet subclasses can use change VisiData Sheet defaults
e. `TsvSheet.option("tsv_optnam", 'default', 'default string for tsv')
   - define option and give it default and helpstr

f. options.set(optname, value, obj='override')
    - `options.unset(optname, obj='override')` to then remove this preference.

`options` is the `OptionsObject` whereas `bindings` and `commands` are `SettingsMgr`s (the OptionsObject references the options `SettingsMgr`).

A `SettingsMgr` stores the value for any key/obj (context) pairs as set by the program and user.

(f) can be useful for commands that toggle options, as set/unset are symmetric.

The default context acts as though it were overridden deliberately by the program at run-time; therefore it is top priority.
This is the same as the options.foo usage.

#### Other methods on SettingsMgr

- `iter(obj=None)`: iterate through all keys in given context (current sheet if None). Yields (key, obj), value.
- `objname(obj)`: return identifiable string for this obj (for use in e.g. cmdlog)
- `getobj(objname)`: get obj in current context from name returned by objname()
- `setdefault(k, v)`: set lowest priority global default (sugar)

Invariant: `getobj(objname(obj)) is obj`

#### Other methods on `options` (the OptionsObject)

i. keys(obj=None): iterate through all optnames with values for given context.
j. getdefault(): get the lowest priority (program definition) default (for e.g. the options sheet)
k. `options('mod\_')`: return dict of ("optname", value) for keys with the prefix 'mod_` (and with the prefix removed).

The dict returned by (k) is designed to be used as kwargs to loaders.
For example, `csv.reader(fp, **options('csv_'))` is how the csv loader passes the csv options transparently to the builtin Python csv module.
The setting values are as they are for the current sheet.

#### global methods

`option('mod_optname', 'default value', 'One line description for generated help/manpages')`

(Use the alias `theme(..)`, which has identical behavior, for cosmetic options that affect neither computation nor output.)

##### `loadConfigFile(fn, _globals=None)`

Use Python `compile()` and `exec()` the given config filename, with access only to the given `\_globals` (or all `globals()` by default).
Add the given globals into the visidata module namespace.

##### vd.parseArgs(parser)

Take existing argparse.ArgumentParser and parse command line arguments to override user configuration file and program defaults.
Ignore theme options, which start with `color\_` or `disp\_`.
Also load config file (as above), and populate the global namespace with the given globals (including any updates and additions).
