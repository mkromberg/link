# Link.Resync 

    ]LINK.Resync <ns>
    
    msg ← {opts} ⎕SE.Link.Resync ns

`Link.Resync` will re-synchronise your workspace and source directories. It is the best way to resume work if you have used [Link.Pause](Link.Pause.md) to temporarily stop watching the file system, or you have loaded a checkpoint workspace that might contain obsolete code, or you have any other reason to suspect that the contents of the active workspace no longer match the source directories.

**WARNING:** Resync is one of the last items of functionality added to Link, and should be considered somewhat experimental in Link 3.0. While this is the case, the default value for the `confirm`  option will be `list` ,  which means that Resync will display output documenting the updates that it intends to make: you will need to explicitly set `confirm=yes`.



The current plan is that, once Resync reaches maturity, an optional [Crawler](Crawler.md) will be made available and the Link version number will become 3.1. The *Crawler* will run Resync in the background, from time to time.



Resync will display a list of differences.

If you had previously used [Link.Pause](Link.Pause.md), your links will no longer be in a paused state following a Resync - unless you set the `pause` option.




#### Arguments

- namespace(s)

#### Options

- **confirm**
  
  > Whether to consider the ns or dir as the authoritative source for the link.
  > - `list` means that a list of actions that would be performed will be displayed.
  > - `yes` means that the actions will be performed.
  > - `copy` means that the actions will be performed and the list of actions will also be returned.
  >
  > Defaults to `list` in 3.0, this is expected to change in Link 3.1.
  
- **pause**

  > Whether the link should be in a paused state following the resync.
  >
  > Defaults to `no`.

#### Result

- String describing the changes made, if any.