# History

Link 3.0, released in 2021, is the one step in a long evolution , developed in 2019

### Workspaces

Historically, APL systems have used *saved workspaces* as the way to store the current state of the interpreter in a binary file, containing all code and data that has been defined in the session. In many ways, a workspace is similar to a workbook saved by a spreadsheet application, a very convenient package that contains everything the application needs.

### Component Files and SQL Databases

Workspaces are very convenient, but the binary format makes them awkward if you want to compare, or otherwise manage different versions of the source code. As people started writing larger systems, many development teams created their own source code management systems, typically storing multiple versions of code in component files or SQL tables.

These SCM's served large developer teams well for several decades. However, none of them became tools that were shared by the APL community, and they all suffered from the fundamental problem of using binary formats.

### SALT - the Simple APL Library Toolkit

In 2006, Dyalog APL Version 11.0 introduced Classes and the ability to represent Namespaces using a text "script". With that release, Dyalog APL included a tool known as [SALT](https://docs.dyalog.com/18.1/SALT%20User%20Guide.pdf), which supported the use of Unicode text files as backing for the source of not only classes and namespaces, but functions and operators as well. SPICE added user commands to Dyalog APL, using text source files which implemented a specific API, based on SALT's file handling.

SALT is Link's direct predecessor, and has many of the same features as Link:

* The ability to load entire directory structures into the workspace as namespaces
* A tool called "Snap", which would write all or selected parts of a workspace to corresponding source files.
* A hook in the APL system editor, which would update source files as soon as code was edited, without requiring a separate save operation.
* Startup processing of files with a `.dyapp` extension, to allow launching applications from text files without requireing a "boot workspace".

### Link 2.0

After SALT had grown organically for more than a decade, it was time for a new start, and Link was born. The first version of Link that was released to the general public was 2.0. The main differences between Link and SALT are:

* Link delegates the task of maintaining information about external source files to the APL interpreter, rather than using a trailing comment in functions and operators or "hidden" namespaces for classes and namespaces to track this information.
  * The new interpreter functionality based on `2 ⎕FIX` makes it possible for the interpreter to preserve source code exactly as typed, when an external source file is used.
* The addition of  a file system watched added support for using external editors and immediately replicating the effect of SCM system actions, such as a git pull or revert operation, inside the active workspace.
* Rather than using the extension `.dyalog` for all source, Link uses different extensions for different types of source, such as `.aplf` for functions, `.apln` for namespaces, and `.apla` for arrays.
* Use of a model of the proposed [Literal Array Notation]() to represent arrays, rather than the notation used by SALT.
  * We hope to add support for the array notation to the Dyalog APL interpreter in a future release.

*  Link has no source code management features; the expectation is that users who require SCM will combine Link with an external SCM such as Git or SVN
  * SALT included a simple mechanism for storing and comparing multiple versions of the source for an object by injecting digits into the file name.

### Link 3.0

Link 3.0 is the first major revision of Link, following real use by a small number of clients. It adds:

* Support for saving workspaces containing linked code and resuming work after a break.  
* Support for names which differ only in case (for example, `FOO` vs `Foo`) in case-insensitive file systems, by adding "case coding" information to the file name.
* A new `Link.LaunchDir` API function, that makes it straightforward to replace the old SALT `.dyapp` files with new features in the interpreter, that make it possible to launch the interpreter using a configuration file or single APL source file.

A more complete description of the differences between Link 2.0 and 3.0 are described in the guide on [upgrading from 2.0 to 3.0](/Upgradeto30.md).

### The Future

The Link road map currently includes the following goals

* Adding the [Crawler](/Crawler.md), which will automatically run [Link.Resync](/API/Link.Resync.md) in the background, in order to detect and help eliminate differences between the contents of linked namespaces and the corresponding directories and can replace the File System Watcher in environments where it is not available.
* Eliminating the use of SALT, with a new implementation of user commands and other mechanisms for loading source code into the interpreter, based on Link rather than SALT.
* Support for linking individual source files. Link 3.0 is only able to link a namespace to a directory. There are situations where it is practical to create a link to a single source file, particularly in the case of a namespace.
* Improving integration with the APL Interpreter so that the editor will honour a Pause.

Over time, it is a strategic goal for Dyalog to move more of the work done by Link into the APL interpreter, such as:

* Serialisation and Deserialisation of arrays, using the literal array notation
* File System Watching or other mechanisms for detecting changes to source at both ends of a link

