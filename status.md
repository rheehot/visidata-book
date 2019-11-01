# The Status Bar

[statusbar.py]

The bottom line of the screen contains the "status bar".
The position of the status bar is not configurable.
However, nearly everything else is.

The bottom line consists of the Left Status, the Status Messages, and the Right Status.

## Left Status: `options.disp_status_fmt`

The leftside status bar holds static information.
By default this includes the sheet name.

## Status Messages

After the Left Status are any status messages that have happened since the last command.
These will linger as long as no key is pressed, and are cleared on the next command.
They can be revisited by opening the StatusSheet with `Ctrl+P` [homage to nethack].

- `options.disp_status_sep`
- `disp_lstatus_max`

### [dev]
- `vd.status(*msgs)`: `color_status`
- `vd.warning(*msgs)`:  `color_warning`
- `vd.fail(msg)`: `color_warning` and raise ExpectedException (bumpers)
- `vd.error(msg)`: `color_error` and raise (internal error)
- `vd.[uncaught]exception(e)`

#### 

error/warning/fail messages are moved to the front of the status message list, since they are highest priority and should not be missed.

## Right Status: `options.disp_rstatus_fmt`

The rightside status bar holds several little pieces about the current state:

- memory usage (if `options.min_memory_mb` set; see safety.md)
- number of rows and rowtype (`options.disp_rstatus_fmt`)
  - universally useful context
- keystrokes from previous command
- processing status (see async.md)
  - percent complete
  - gerund
- replay status: `color_status_replay` (see replay.md)
  - step N/total
  - playing, paused, or stopped

## [dev]

BaseSheet.leftStatus() and BaseSheet.rightStatus() return a string, with always-visible status information about the current sheet.
By default they return the formatted options.

These can be overridden by subsheets, though if the same display can be
accomplished by creating additional properties on the Sheet to be used in the
option override, then that should be preferred.

---
