# Link.Create

    ]LINK.Create <ns> <dir> [-source={ns|dir|auto}] [-watch={none|ns|dir|both}] [-casecode] [-forceextensions] [-forcefilenames] [-arrays] [-sysvars] [-flatten] [-beforeread=<fn>] [-beforewrite=<fn>] [-getfilename=<fn>] [-codeextensions=<var>] [-typeextensions=<var>] [-fastload] 
    
    msg ← {opts} ⎕SE.Link.Create (ns dir)

#### Arguments

- **namespace** name of - or reference to - a namespace
- **directory** name of a file system directory (without trailing slash or backslash)

#### Result

- String describing the established link, along with possible failures

#### Common Options

- **source**	{ns|dir|**auto**}  
  
  > Whether to consider the ns or dir as the source (also used by a subsequent [Refresh](Link.Refresh.md)).
  > - `dir` means that the namespace must be non-existent or empty and will be populated from source files.
  > - `ns` means that the directory must be non-existent or empty and will be populated by source files for the items in the namespace.
  > - `auto` will use whichever of ns or dir that is not empty. If both are empty, it will use `dir` on a subsequent [Refresh](Link.Refresh.md).
  >
  > Defaults to `auto`.
  
 - **watch**	{none|ns|dir|**both**} 
  > Specifies which sides of the link to watch for changes (and synchronise).
  > - `ns` will mirror namespace changes (done with the editor) to files. Note that it will **not** reflect changes made using other mechanisms, such as assignment, `⎕FX`, `⎕FIX`, `⎕CY`,  or `⎕NS`. If you want to programmatically change an item so that the change is reflected to files, you should use [⎕SE.Link.Fix](Link.Fix.md).
  > - `dir` will mirror changes made to files (using any mechanism) into the namespace. Note that there is a chance that massive file changes (e.g. git checkout, git pull or an unzip) may cause the file system watcher to miss changes. It is recommended to [Link.Pause](Link.Pause.md) the link before doing massive changes to files, then [Link.Resync](Link.Resync.md) to resume file watching.
  > - `both` will do both.
  >
  > Watching a `dir` (or `both`) is currently only supported using the .Net Framework or .NetCore, but a [Crawler](/Crawler.md) is planned to perform watching on all platforms, and to recover from cases where changes are not picked up by other mechanisms.
  > 
  > The default is `both` where supported, else `ns`.  

- **caseCode** (default off) Adds a suffix to file names on write
  > If your application contains items with names that differ only in case (for example `Debug` and `DEBUG`), and your file system i s case-insensitive (for example, under Microsoft Windows), then enabling **caseCode** will cause a suffix to be added to file names, containing an octal encoding of the location of uppercase letters in the name.
  > 
  > For example, with caseCode on, two functions named `Debug` and `DEBUG` will be written to files named
  > `Debug-1.aplf` and `DEBUG-37.aplf`.
  > 
  >Note: Dyalog recommends that you avoid creating systems with names that differ only in case. This feature primarily exists to support the import of applications which already use such names. Also note that you will probably want to enable **forceFilenames** if you enable **caseCode**.
  
- **forceExtensions** (default off) Force correct extensions
  
  > If enabled, file extensions will be adjusted (if necessary) when an item is defined in the workspace from an external file, so that the file extension accurately reflects the type of the item according to **typeExtensions**.

- **forceFilenames** (default off) Force correct filenames
  
  > If enabled, file names will be adjusted so that they match the item name, when an item is defined 
  > in the workspace from an external file, so that the file name matches the name of the item.
  >
  > By default, Link will always new files with the same name as items created in the active workspace. However, it will not insist that file names match item names when importing items from a directory.
  > 
  > If **forceFilenames** is not set.  Link will update to the same file that an item was loaded from, even though the file name does not match the item name.

- **arrays** (default off) Export arrays
  
   > - if simply set (to 1) (e.g. `-arrays`), then all arrays are exported 
   > - if set to a comma-separated list of names (e.g. `-arrays=name1{,name2,...}`) then arrays with specified names are exported
   >
> This option takes effect only when **source** is **ns**, and only when the link is initially created. Arrays will not be monitored for changes during operation of the application.

- **sysVars** (default off) Export namespace-scoped system variables to file
  
  > The exhaustive list of exported variables is: `⎕AVU  ⎕CT  ⎕DCT  ⎕DIV  ⎕FR  ⎕IO  ⎕ML  ⎕PP  ⎕RL  ⎕RTL  ⎕USING  ⎕WX`. They will be exported for all unscripted namespaces.
  >
  > This option takes effect only when **source** is **ns**.

#### "Advanced" Options

- **flatten** (default off) Do not create sub-namespaces
  > **flatten** will load all items into the root of the linked namespace, even if the source code is arranged into sub-directories. This is typically used for applications which have source which is divided into modules, but still expects to run in a "flat" workspace.
  > 
  > Note that if **flatten** is set, new items need special treatment:
  > - If a function or operator is renamed in the editor, the new item will be placed in the
  >same folder as the original item.
  > - If a new item is created, it will be placed in the root of the linked directory.
  > - It is also possible to use the **getFilename** setting to add application-specific logic to determine the file name to be used (or prompt the user for a decision).
  > - A simple work-around is to always create a stub source file in the correct directory and editing the function that appears in the workspace, rather than creating new functions in the workspace.
  > 
  > This option takes effect only when **source** is **dir**.


- **beforeWrite** `ns.hookname` name of function to call before writing to file

  > If you specify a **beforeWrite** function, it will be called before Link updates a file or directory, allowing support of custom code or data formats. 
  >
  > Your function will be called with a nested right argument containing the following elements:
  >
  > |Index|Description|
  > | ---- | ---- |
  > |[1]|Event name ('beforeWrite')|
  > |[2]|Reference to a namespace containing link options for the active link.|
  > |[3]|Fully qualified filename that Link intends to write to (directories end with a slash)|
  > |[4]|Fully qualified APL name of the item that Link intends to write|
  > |[5]|Name class of the APL item to write|
  > |[6]|Old APL name (different from APL name if the write is due to a rename)|
  > |[7]|Source code that Link intends to write to file|
  >
  > Note: Do not assume a specific length, more elements may be added in the future.
  >
  > Your callback function must return one of the following results:
  >
  >  - `0`: The **beforeWrite** function has completed all necessary actions. Link should not update any files.
  >  - `1`: The **beforeWrite** function wishes to "pass" on this write: Link should proceed as planned.

- **beforeRead** `ns.hookname` name of function to call before before reading a file

  > If you specify a **beforeRead** function, it will be called before Link reads source from a file or directory, allowing support of custom code or data formats.
  > 
  > Your function will be called with a nested right argument containing the following elements:
  > 
  > |Index|Description|
  > | ---- | ---- |
  > |[1]|Event name (`'beforeRead'`)|
  > |[2]|Reference to a namespace containing link options for the active link.|
  > |[3]|Fully qualified filename that Link intends to read from (directories end with a slash)|
  > |[4]|Fully qualified APL name of the item that Link intends to update|
  > |[5]|Name class of the APL item to be read|
  > 
  > Note: Do not assume a specific length, more elements may be added in the future.
  > 
  >Your callback function must return one of the following results:
  >  - `0`: The **beforeRead** function has completed all necessary actions. Link should not update the workspace.
  >  - `1`: The **beforeRead** function wishes to "pass" on this read: Link should proceed as planned.

- **getFilename** `ns.hookname` name of the function to call to decide the file or directory name linked to an APL item
  > If you specify a **getFilename** function, it will be called before Link updates a file or directory, allowing you to modify the name (or more likely the extension) of the file used to store the source for an APL item. Changing the file name this way allows you to override the **caseCode**, **forceFilenames** and **forceExtensions** options.
  >
  > Your function will be called with a nested right argument containing the following elements:
  > |Index|Description|
  > | ---- | ---- |
  > |[1]|Event name (`'getFilename'`)|
  > |[2]|Reference to a namespace containing link options for the active link.|
  > |[3]|Fully qualified filename that Link intends to use (directories end with a slash)|
  > |[4]|Fully qualified APL name of the item|
  > |[5]|Name class of the APL item|
  > |[6]|Old APL name (different from APL name if the write is due to a rename)|
  > 
  > Note: Do not assume a specific length, more elements may be added in the future.
  > 
  > Your callback function must return a character vector which must be:
  >  - empty: to signify that Link should proceed with the suggested file name.
  >  - non-empty: to specify the name to be used.
  
- **codeExtensions** File extensions that are expected to contain source code
  
  > When reacting to changes in a watched directory, Link will only process files if the changed file has one of the listed extensions.
  > 
  > The default is `'aplf' 'aplo' 'apln' 'aplc' 'apli' 'dyalog' 'apl' 'mipage'`
  > 
  >From a user command, the syntax is `-codeExtensions=var` where `var` holds the expected array of extensions.

- **customExtensions** Specifies additional file extensions handled by **beforeRead** functions
  > If you have specified a **beforeRead** handler function, and your code supports the use of custom file extensions to store source data in application-specific formats, you need to set **customExtensions** so that Link does not ignore changes to these file types.
  > 
  > default is `''` - no custom extensions
  > 
  >From a user command, the syntax is `-customExtensions=var` where `var` holds the expected array.
  > 
  >The reason for splitting the list of extensions into two parts is to avoid your code having to repeat the list of standard extensions, or update this list if it should be extended in the future.
  
- **typeExtensions** Specify the default file extensions to use when creating files
  
  > The **typeExtensions** table specifies the default extension that should be used when creating a new file to contain the source for an item of a given type. 
  > 
  > **typeExtensions** is a two-column matrix with numeric name class numbers in the first column
  >and corresponding file extensions in the second column.
  > 
  > Note that the **forceExtensions** switch can be used to correct all extensions on pre-existing files when a link is created.
  >
  > The default is:
  > 
  >| Type | extension |
  > | ---- | --------- |
  >| 2    | apla      |
  > | 3    | aplf      |
  > | 4    | aplo      |
  > | 9.1  | apln      |
  > | 9.4  | aplc      |
  > | 9.5  | apli      |
  > 
  > From a user command, the syntax is `-typeExtensions=var` where `var` holds the expected array.


- **fastLoad** (default off) Flag to reduce the load time by not inspecting source to detect name clashes
  >   This affects only initial directory loading, but not subsequent editor or file system watcher events. It is worth setting fastLoad for very large projects with users that don't produce name clashes (i.e. two files defining the same APL name). 
  >
  > Side effects are (again, only at initial load time, not at subsequent events):
  >   - good: load will be significantly faster because files won't be inspected to determine their true APL name.
  >   - bad: clashing names won't be detected: files may silently overwrite each other's APL definition if they define the same APL name.
  >   - bad: **forceFileNames**/**forceExtensions** won't be observed
  >   - bad: **beforeRead** may report incorrect name class
  >
  > This option takes effect only when **source** is **dir**.
