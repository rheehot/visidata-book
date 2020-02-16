The VisiData class is a singleton, containing all of the 'global' state for the current session. Currently, it must be available in the `vd` global. Eventually, it might be substitutable for perhaps multiple shared sessions.

The **Sheet** and **VisiData** class are both in scope for execstrs, making them home for functions and properties that should be easily callable by commands.

`from visidata import vd`, then the global API would be everything on the VisiData object.

#### `@drawcache_property`

Works like `Extensible.@cached_property` but will call `clear_cache()` just before the end `draw()`.
Used for properties that are expected not to change within a draw cycle, and will be computer multiple times per `draw()`.

### `save_ext()`

`save_ext()` functions are called when users want to save files with the provide *ext*. They are callable through the VisiData class.

The declaration for them takes the form of:

```
@Visidata.api
def save_ext(vs, p, *sheets)
```

where `p` is a `visidata.Path()` object representing the file being written to, and `*sheets` will be the 1 sheet or more that are going to be saved.
