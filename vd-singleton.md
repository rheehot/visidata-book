The VisiData class is a singleton, containing all of the 'global' state for the current session. Currently, it must be available in the `vd` global. Eventually, it might be substitutable for perhaps multiple shared sessions.

`from visidata import vd`, then the global API would be everything on the VisiData object.

#### `@drawcache_property`

Works like `Extensible.@cached_property` but will call `clear_cache()` just before the end `draw()`.
Used for properties that are expected not to change within a draw cycle, and will be computer multiple times per `draw()`.
