# Technical Details and Limitations

Link enables the use of text files as application source, by mapping workspace content to directories and files. There are some types of objects that cannot be supported, and a few other limitations that are worth discussing:

## Supported Objects

In the following, for want of a better word, the term *object* will be used to refer to any kind of named entity that can exist in an APL workspace - not limited to classes or instances.

**Supported:** Link supports objects of name class 2.1 (array), 3.1 (traditional function), 3.2 (d-function), 4.1 (traditional operator), 4.2 (d-operator), 9.1 (namespace), 9.4 (class) and 9.5 (interface).

**Unscripted Namespaces:** Namespaces created with `⎕NS` or `)NS` have no source code of their own and are mapped to directories. In Link version 3.0, one endpoint of a link is always an unscripted namespace, and the other endpoint of the link is a directory.

**Scripted Namespaces:** So-called scripted namespaces, created using the editor or `⎕FIX`, have textual source and are treated the same way as functions and other "code objects". Link 3.0 does not support scripted namespaces, or any other objects that map to a single source file, as endpoints for a link.

It is likely that this restriction will be lifted in a future version of Link.

**Variables** are ignored by default, because most of them are not part of the source code of an application. However, they may be explicitly saved to file with [Link.Add](/API/Link.Add.md), or with the `-arrays` modifier of [Link.Create](/API/Link.Create.md) and [Link.Export](/API/Link.Export.md).

**Functions and Operators:** Link is not able to represent names which refer to primitive or derived functions or operators, or trains - because the APL interpreter does not have a source form for such items. You will need to define such objects in the source of another function, or a scripted namespace.

**Unsupported:** Link has no support for name classes 2.2 (field), 2.3 (property), 2.6 (external/shared variable), 3.3 (primitive or derived function or train), 4.3 (primitive or derived operator), 3.6 (external function) 9.2 (instance), 9.6 (external class) and 9.7 (external interface).

## Other Limitations

* Namespaces must be named. To be precise, it must be true that `ns≡(⎕NS⍬)⍎⍕ns`. Scripted namespaces must not be anonymous. When creating an unscripted namespace, we recommend using `⎕NS` dyadically to name the created namespace (for example `'myproject' ⎕NS ⍬` rather than `myproject←⎕NS ⍬`). This allows retrieving namespace reference from its display from (for example `#.myproject` rather than `#.[namespace]`).
* Link does not support namespace-tagged functions and operators (e.g. `foo←namespace.{function}`).
* Changes made using `←`, `⎕NS`, `⎕FX`, `⎕FIX`, `⎕CY`, `)NS` and `)COPY` or the APL line `∇` editor are not currently detected. For Link to be aware of the change, a call must be made to [Link.Fix](/API/Link.Fix.md). Similarly, deletions with `⎕EX` or `)ERASE` must be replaced by a call to [Link.Expunge](/API/Link.Expunge.md).
* Link does not support source files that define multiple names, even though 2∘⎕FIX does support this.
* The detection of external changes to files and directories is currently only supported under .Net and .Net Core. Note that the built-in APL editor *will* detect changes to source files on all platforms, but not before the editor is opened.
* Source code must not have embedded newlines within character constants, although ⎕FX does allow this. Link will error if this is attempted. This restriction comes because newline characters would be interpreted as a new line when saved as text file. When newline characters are needed in source code, they should be implemented by a call to `⎕UCS` e.g. `newline←⎕UCS 13 10  ⍝ carriage-return + line-feed`
* Although Link 3.0 will work with version 18.0, Dyalog v18.1 is recommended if it is important that all source be preserved as typed. Earlier versions of APL have small glitches that occasionally lose the source as typed under certain circumstances.
