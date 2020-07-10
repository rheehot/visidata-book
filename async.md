# Asynchronous processing (visidata.threads)

## Maintaining a responsive interface

VisiData does not do the fastest calculations, but great care has been taken to make sure that it remains *responsive*.
It is better to wait 10 seconds with active indication of progress, than 2 seconds with the program frozen and not accepting input (worse; queueing it up for later without ability to cancel).

To this end, VisiData has a main display/input thread, and creates threads for any task that may be time-consuming.
In fact, anything that iterates through all rows (there may be millions) or all columns (there may be thousands), will probably be in its own thread.


## [dev] `@asyncthread`

Add the `@asyncthread` decorator to a function, so calls to it will spawn a new thread.
The return value is the spawned thread (which can often be ignored by the caller); the return value of the original function is effectively lost.

Note, that if you chain decorators, `@asyncthread` should be the closest decorator to the function. Otherwise, `@asyncthread` will spawn a new thread for the *act of decorating*, and not for the function call itself.

Cells which are being computed in a separate thread should have that thread as their value until their result is available.
This will show the `options.disp_pending` notation and allow the user to interact with the specific thread (via e.g. `z^Y` and others).

To call the original form of an asyncthread function, call `func.__wrapped__(self)`.

Each thread is added to `Sheet.currentThreads` for the current sheet.
Note that a thread spawned by calling a function on a different sheet will add the thread to the currentThreads for the topmost/current sheet instead.

### Canceling threads

The user can cancel all `Sheet.currentThreads` with `Ctrl+C`.

Internally, `cancelThread(*threads)` will send each thread an `EscapeException`, which percolates up the stack to be caught by the thread entry point.
EscapeException inherits from BaseException instead of Exception, so that threads can still have catch-all try blocks with `except Exception:`.
An unqualified `except:` clause is bad practice (as always); when used in an async function, it will make the thread uncancelable.

### [dev] Wait for threads to finish

`sync(*joiningThreads)` will wait for all but some number of `expectedThreads` to finish.

This may be useful for a function f1() which calls an @asyncthread f2(), and then does something afterwards.  f1 should also be asyncthread, and call `sync(f2())` (since f2 will return its thread object).

sync() without any parameters waits for all threads except the main thread and the current thread to finish.

# Threads Sheet (`Ctrl+T`/`threads-all`)

All threads (active, aborted, and completed) are added to `VisiData.threads`, which can be viewed as the ThreadsSheet via `Ctrl+T`.
Threads which take less than `min_thread_time_s` (hardcoded in `asyncthread.py` to 10ms) are removed, to reduce clutter.

- set `options.profile` to a filename prifix to store profile datawas True when the thread started).

## options
option('min_memory_mb', 0, 'minimum memory to continue loading and async processing')
theme('color_working', 'green', 'color of system running smoothly')

## Commands

- `Ctrl+_`/`toggle-profile`: toggle profiling of the main thread, and also set `options.profile` to `vdprofile` (hard-coded).
- [Threads Sheet] `Enter` view the performance profile for the current thread.
  - [Profile Sheet] `z Ctrl+S`/`save-profile`: 
ProfileSheet.addCommand('z^S', 'save-profile', 'source.dump_stats(input("save profile to: ", value=name+".prof"))')
ProfileSheet.addCommand(ENTER, 'dive-row', 'vd.push(ProfileSheet(codestr(cursorRow.code)+"_calls", source=cursorRow.calls or fail("no calls")))')
ProfileSheet.addCommand('z'+ENTER, 'dive-cell', 'vd.push(ProfileSheet(codestr(cursorRow.code)+"_"+cursorCol.name, source=cursorValue or fail("no callers")))')
ProfileSheet.addCommand('^O', 'sysopen-row', 'launchEditor(cursorRow.code.co_filename, "+%s" % cursorRow.code.co_firstlineno)')
globalCommand('^_', 'toggle-profile', 'toggleProfiling(threading.current_thread())')

BaseSheet.addCommand('^C', 'cancel-sheet', 'cancelThread(*sheet.currentThreads or fail("no active threads on this sheet"))')
globalCommand('g^C', 'cancel-all', 'liveThreads=list(t for vs in vd.sheets for t in vs.currentThreads); cancelThread(*liveThreads); status("canceled %s threads" % len(liveThreads))')
globalCommand('^T', 'threads-all', 'vd.push(vd.threadsSheet)')
## Profiling

The view of a performance profile in VisiData is the output from `pstats.Stats.print_stats()`.

- `z^S` on the performance profile will call `dump_stats()` and save the profile data to the given filename, for analysis with e.g. [pyprof2calltree]() and [kcachegrind]().
- (`z^S` because the raw text can be saved with `^S` as usual.  Ideally, `^S` to a file with a `.pyprof` extension on a profile sheet would do this instead.)

# Progress counters

In all `@asyncthread` functions, a `Progress` counter should be used to provide a progress percentage, which appears in the right-hand status.

## Progress as iterable

When iterating over a potentially large sequence:

    for item in Progress(iterable, gerund='working'):

This is just like `for item in iterable`, but it also keeps track of progress, to display on the right status line.  The `gerund` specifies the action being performed.

- This only displays if used in another thread, and has no effect otherwise; so any linear action should have Progress, even if it is not @asyncthread.
- Use Progress around the innermost iterations for maximum granularity and apparent responsiveness.
- But this incurs a small amount of overhead, so if a tight loop needs every last optimization, use it with an outer iterator instead (if there is one).
- Multiple Progress objects used in parallel will stack properly.
- Multiple Progress objects used serially will make the progress indicator reset (which is better than having no indicator at all).

If `iterable` does not know its own length, it (or an approximation) should be passed as the `total` keyword argument:

    for item in Progress(iterable, total=approx_size):

The `Progress` object contributes 1 towards the total for each iteration.
To contribute a different amount, use `Progress.addProgress(n)` (n-1 if being used as an iterable, as 1 will be added automatically).

If the progressTotal is 0, then the percentage in the righthand status bar is omitted (the gerund is still shown).

## Progress as context manager

To manage `Progress` without wrapping an iterable, use it as a context manager with only a `total` keyword argument, and call `addProgress` as progress is made:

    with Progress(total=amt) as prog:
        while amt > 0:
            some_amount = some_work()
            prog.addProgress(some_amount)
            amt -= some_amount

- Using `Progress()` other than as an iterable or a context manager will have no effect.

## properties

## Sheets

### ThreadsSheet

- Command|`open-threads`|'Ctrl+T'|open Threads Sheet|meta|

### ProfileSheet

### BaseSheet

`progressMade`
`progressTotal`

# 
`@asyncsingle`
`@asynccache`

### VisiData methods/properties
- `vd.sync`
- `vd.threads`
- `vd.threadsSheet`
- `vd.unfinishedThreads`
- `vd.checkForFinishedThreads`
- `vd.cancelThread`
- `vd.execAsync`


.pyprof

---
__all__ = ['Progress', 'asynccache', 'async_deepcopy', 'elapsed_s', 'cancelThread', 'ThreadsSheet', 'ProfileSheet', 'codestr', 'asyncsingle']
