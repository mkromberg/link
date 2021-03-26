# Link.Create

    ]LINK.Create <ns> <dir> [-source={ns|dir|auto}] [-watch={none|ns|dir|both}] [-casecode] [-forceextensions] [-forcefilenames] [-arrays] [-sysvars] [-flatten] [-beforeread=<fn>] [-beforewrite=<fn>] [-getfilename=<fn>] [-codeextensions=<var>] [-typeextensions=<var>] [-fastload] 

    msg ← {opts} ⎕SE.Link.Create (ns dir)

#### Arguments

- **namespace** name of, or reference to a namespace
- **directory** name of a file system directory (without trailing slash or backslash — the plan is to let these indicate that the directory name is to be inferred from the namespace name)

#### Result

- String describing the established link, along with possible failures

#### Common Options

- **source**	{ns|dir|**auto**}  
  > Whether to consider the ns or dir as the authoritative source at [Create](Link.Create.md) and [Refresh](Link.Refresh.md) time.
  > - `dir` means that the namespace must be non-existent or empty and will be overwritten by items in files.
  > - `ns` means that the directory must be non-existent or empty and will be overwritten by items in the namespace.
  > - `auto` will use whichever of ns or dir that is not empty. If both are empty, it will use `dir`.
  >
  > \
  > Defaults to `auto`.

 - **watch**	{none|ns|dir|**both**} 
   > Specifies which sides of the link to watch for changes (and synchronise).
   > - `ns` will mirror namespace changes (done with the editor) to files. Note that it will **not** reflect changes made by running code (e.g. using assignment, `⎕FX`, `⎕FIX`, `⎕CY`, `⎕NS`, etc). If you want to programmatically change an item so that the change is reflected to files, you need to use [⎕SE.Link.Fix](Link.Fix.md)
   > - `dir` will mirror file changes (done with any software) into the namespace. Note that massive file changes (e.g. git checkout or git pull) may fail and leave the link in an unsynchronised state, in which case you will get a warning message. Therefore it is desirable to [Pause](Link.Pause.md) the link before doing massive changes to files, then [Refresh](Link.Refresh.md) the link to resume file watching.
   > - `both` will do both.
   >
   > \
   > Watching a `dir` (or `both`) is currently only supported using the .Net Framework or .NetCore, but a fully cross-platform support is planned.
   > Defaults to `both` where supported, else `ns`.  

- **caseCode** (default off) Adds a suffix to file names on write
  > If your application contains items with names that differ only in case
  > (for example `Debug` and `DEBUG`), and your file system is case-insensitive
  > (for example, under Microsoft Windows), then enabling **caseCode** will
  > cause a suffix to be added to file names, containing
  > an octal encoding of the location of uppercase letters in the name.
  >
  > For example, with caseCode on, two functions named `Debug` and `DEBUG` will be written to files named
  > `Debug-1.aplf` and `DEBUG-37.aplf`.
  >
  > Note: you will probably want to enable **forceFilenames** if you enable **caseCode**.

- **forceExtensions** (default off) Force correct extensions
  > If enabled, file extensions will be renamed when an item is defined in the workspace from an external file,
  > so that the file extension accurately reflects the type of the item according to **typeExtensions**.

- **forceFilenames** (default off) Force correct filenames
  > If enabled, file names will be adjusted so that they match the item name, when an item is defined 
  > in the workspace from an external file, so that the file extension accurately reflects the name of the item.
  >
  > Note that by default, although Link will always create files with the same name as items added to the
  > active workspace, it will not insist that file names match item names when importing items from a directory.
  > Unless **forceFilenames** is set, Link will write updates to the same file that an item was loaded from,
  > even though the file name does not match the item name.

- **arrays** (default off) Allow exporting arrays too
   > - if simply set (to 1) (e.g. `-arrays`), then all arrays are exported 
   > - if set to a comma-separated list of names (e.g. `-arrays=name1{,name2,...}`) then arrays with specified names are exported
   >
   > This option takes effect only when **source** is **ns**.

- **sysVars** (default off) Export namespace-scoped system variables to file
  > The exhaustive list of exported variables is: `⎕AVU  ⎕CT  ⎕DCT  ⎕DIV  ⎕FR  ⎕IO  ⎕ML  ⎕PP  ⎕RL  ⎕RTL  ⎕USING  ⎕WX`. They will be exported for all unscripted namespaces.
  >
  > This option takes effect only when **source** is **ns**.

#### "Advanced" Options

- **flatten** (default off) Do not create sub-namespaces
  > **flatten** will load all items into the root of the linked namespace, without creating any sub-namespaces,
  > even if the source code is arranged into sub-directories. This is typically used for old applications
  > which have been divided into source modules, but still need
  > all code to be loaded into a single namespace.
  >
  > Note that if **flatten** is set, new items need special treatment:
  > - If a function or operator is renamed in the editor, the new item will be placed in the
  > same folder as the original item.
  > - If a new item is created, it will be placed in the root of the linked directory.
  > - It is also possible to use the **getFilename** setting to add application-specific logic to determine the file name to be used.
  >
  > This option takes effect only when **source** is **dir**.


- **beforeWrite** `ns.hookname` name of function to call before writing to file

  > If you specify a **beforeWrite** function, it will be called before Link updates a file or directory, allowing support of custom code or data formats. 
  >
  > Your function will be called with a nested right argument containing the following elements:\
  > [1] Event name (`'beforeWrite'`)\
  > [2] Reference to a namespace containing link options for the active link.\
  > [3] Fully qualified filename that Link intends to write to (directories end with a slash)\
  > [4] Fully qualified APL name of the item that Link intends to write\
  > [5] Name class of the APL item to write\
  > [6] Old APL name (different from APL name if the write is due to a rename)\
  > [7] Source code that Link intends to write to file\
  > Note: Do not assume a specific length, more elements may be added in the future.\
  > \
  > Your callback function must return one of the following results:
  >  - `0`: The **beforeWrite** function has completed all necessary actions. Link should not update any files.
  >  - `1`: The **beforeWrite** function wishes to "pass" on this write: Link should proceed as planned.

- **beforeRead** `ns.hookname` name of function to call before before reading a file

  > If you specify a **beforeRead** function, it will be called before Link reads source from a file or directory, allowing support of custom code or data formats.
  > 
  > Your function will be called with a nested right argument containing the following elements:\
  > [1] Event name (`'beforeRead'`)\
  > [2] Reference to a namespace containing link options for the active link.\
  > [3] Fully qualified filename that Link intends to read from (directories end with a slash)\
  > [4] Fully qualified APL name of the item that Link intends to update\
  > [5] Name class of the APL item to be read\
  > Note: Do not assume a specific length, more elements may be added in the future.\
  > \
  >Your callback function must return one of the following results:
  >  - `0`: The **beforeRead** function has completed all necessary actions. Link should not update the workspace.
  >  - `1`: The **beforeRead** function wishes to "pass" on this read: Link should proceed as planned.

- **getFilename** `ns.hookname` name of the function to call to decide the file or directory name linked to an APL item
  > If you specify a **getFilename** function, it will be called before Link updates a file or directory, allowing to customise the name attached to an APL item. Changing the file name this way will override the **caseCode**, **forceFilenames** and **forceExtensions** options (however the suggested file name will observe them)
  >
  > Your function will be called with a nested right argument containing the following elements:\
  > [1] Event name (`'getFilename'`)\
  > [2] Reference to a namespace containing link options for the active link.\
  > [3] Fully qualified filename that Link intends to use (directories end with a slash)\
  > [4] Fully qualified APL name of the item\
  > [5] Name class of the APL item\
  > [6] Old APL name (different from APL name if the write is due to a rename)\
  > Note: Do not assume a specific length, more elements may be added in the future.\
  > \
  > Your callback function must return a character vector which must be:
  >  - empty: to signify that Link should use the intended file name.
  >  - non-empty: to specify which filename must be used by Link.

- **codeExtensions** File extensions that are expected to contain source code
  > When reacting to changes in a watched directory, Link will only process files
  > if the changed file has one of the listed extensions.
  > 
  > The default is `'aplf' 'aplo' 'apln' 'aplc' 'apli' 'dyalog' 'apl' 'mipage'`
  >
  > From a user command, the syntax is `-codeExtensions=var` where `var` holds the expected array.

- **customExtensions** Specifies additional file extensions handled by **beforeRead** functions
  > If you have specified a **beforeRead** handler function, and your code
  > supports the use of custom file extensions to store source data in application-specific
  > formats, you need to set **customExtensions** so that Link does not ignore changes
  > to these file types.
  >
  > default is `''` - no custom extensions
  >
  > From a user command, the syntax is `-customExtensions=var` where `var` holds the expected array.

- **typeExtensions** Specify the file extensions to use for each name class
  > The **typeExtensions** table specifies the default extension
  > that should be used when creating a new file to contain the
  > source for an item of a given type. 
  >
  > **typeExtensions** is a two-column matrix with numeric name class numbers in the first column
  > and corresponding file extensions in the second column.
  >
  > Note that the **forceExtensions**
  > switch can be used to correct all extensions on pre-existing files when a link is created.
  >
  > The default is:
  >
  > | Type | extension |
  > | ---- | --------- |
  > | 2    | apla      |
  > | 3    | aplf      |
  > | 4    | aplo      |
  > | 9.1  | apln      |
  > | 9.4  | aplc      |
  > | 9.5  | apli      |
  >
  > From a user command, the syntax is `-typeExtensions=var` where `var` holds the expected array.


- **fastLoad** (default off) Flag to reduce the load time by not inspecting source to detect name clashes
  >   This affects only initial directory loading, but not subsequent editor or file system watcher events. Worth doing for very large projects with users that don't produce name clashes (i.e. two files defining the same APL name). 
  >
  > Side effects are (again, only at initial load time, not at subsequent events):
  >   - good: load will be significantly faster because files won't be inspected to determine their true APL name.
  >   - bad: clashing names won't be detected: files may silently overwrite each other's APL definition if they define the same APL name.
  >   - bad: **forceFileNames**/**forceExtensions** won't be observed
  >   - bad: **beforeRead** may report incorrect name class
  >
  > This option takes effect only when **source** is **dir**.
