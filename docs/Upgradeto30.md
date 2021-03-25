## Upgrading to Link 3.0

If you are upgrading from Link 2.0 to 3.0, there are many new options and API functions (and corresponding user commands) that are available. The most significant changes are described here.

### Breaking Changes

Some of the changes have the potential to break existing applications that use Link and require a review of existing code that calls the [Link API](API.md):

* When specifying the name of a directory, it may not have a trailing slash (this is reserved for possible future extensions)
* The `source=both` option has been removed from [Link.Create](Link.Create.md).
* Link.List has been renamed [Link.Status](Link.Status.md)
* When providing a new value for an array using [Link.Fix](Link.Fix.md), Link 3.0 expects the text source form of the array rather than the value of the array (as is the case for all other items). To update using the value, assign it to the array and call [Link.Add](Link.Add.md).
* If you have defined handlers for custom array representations, there have been significant changes to the arguments to the `beforeRead` and `beforeWrite`callback functions - and the addition of a new `getFilename` callback. These functions are described in the documentation for [Link.Create](Link.Create.md).

### Other Significant Changes

The most important new features are:

* The addition of a [Crawler](Crawler.md), which will detect and help eliminate differences between the contents of linked namespaces and the corresponding directories.
* The addition of [Link.Pause](Link.Pause.md) and corresponding enhancements to [Link.Refresh](Link.Refresh.md) which provide better support for resuming work after a break, especially if the active workspace has been saved and reloaded during the break.  
* "Case Coding" of file names, supporting the maintenence of source for names which differ only in case (for example, `FOO` vs `Foo`) in case-insensitive file systems.
* The addition of the `Link.LaunchDir` API function, which returns the name of the directory that the interpreter was started from, either using the `LOAD=` or `CONFIGFILE=`setting.

### Release Notes

A detailed list of new features added to recent releases and a few behavioural changes can be found in the [Link 3.0 Release Notes](ReleaseNotes30.md). 



