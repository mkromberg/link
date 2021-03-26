# Technical Details and Limitations

Link supports the use of text files as application source, by mapping workspace content and filesystem content in a one-to-one relationship. Each unscripted namespace is represented as a directory. Defined functions, scripted namespaces, classes and interfaces and arrays are associated with a text file. Depending on which synchronisation options are available and have been selected, Link will synchronise changes in none, one or both directions.

## Supported Objects

In the following, for want of a better work, the term *object* will be used to refer to any kind of named entity that can exist in an APL workspace - not limited to classes or instances.

Link handles objects of name class 2.1 (array), 3.1 (traditional function), 3.2 (d-function), 4.1 (traditional operator), 4.2 (d-operator), 9.1 (namespace), 9.4 (class) and 9.5 (interfaces).

Unscripted namespaces (created with ⎕NS or )NS and not :Namespace or :Class or :Interface) are mapped to directories. Functions, operators and namespaces that have text source are mapped to text files.

Variables are ignored by default, but they may be explicitly linked and saved to file with [Link.Add](/API/Link.Add.md), or with the `-arrays` modifier for [Link.Create](/API/Link.Create.md) and [Link.Export](/API/Link.Export.md).

* The root namespace of a link must be an unscripted namespace (created with ⎕NS)
* Link does not support name classes 2.2 (field), 2.3 (property), 2.6 (external/shared variable), 3.3 (primitive or derived function or train), 4.3 (primitive or derived operator), 3.6 (external function) 9.2 (instance), 9.6 (external class) and 9.7 (external interface).

By default, Link will update files with changes made in linked namespaces through the editor (`⎕ED`). It will also watch the file system for changes to the linked directory, modifying the linked namespace accordingly. Watching the file system is currently supported only on .Net and .NetCore, but support is planned for other operating systems in the near future.


## Syntax

Link provides a set of utility functions in the namespace `⎕SE.Link` (see [the API reference](/API/index.md) for a list). If SALT and user commands are enabled then an additional `LINK` group of user commands are available (and can be listed with `]?link`)

For performance and ease of debugging, we recommend avoiding the invocation of user commands under program control. Use the utility functions located in `⎕SE.Link` directly instead.

### User commands

The corresponding user commands have the general syntax
```
]LINK.CmdName arg1 [arg2] [-name[=value] ...]
```
where `arg2`'s presence depends on the specific command, and `-name` is a flag enabling the specific option and `-name=value` sets the specific option to a specific value. The value of the two options requiring array values (`codeExtensions` and `typeExtensions`) are instead the *name* of a variable containing the needed array.

### Utility functions

The general syntax of the utility functions is

```apl
options FnName arg
```
where `options` is a namespace with variables named according to the option they set, containing their corresponding values. The `-name=value` option can be set by `options.name←value`, and switches with values (e.g. `-name`) can be set by `options.name←1`.

Unset options take their default value (just like omitted modifiers in the user command).

`arg` is detailed in each function help: see the [API index](/API/index.md)


## Usage

In most cases, it is unnecessary to use any custom options. Link will infer what to do in the first three of the following four cases:

1. The namespace is empty or doesn't exist and the directory is empty or doesn't exist.
1. The namespace is empty or doesn't exist but the directory has content
1. The namespace has content but the directory is empty or doesn't exist
1. The namespace has content and the directory has content

In the first three cases (where no more than one side has content), Link will export/import any content to the other side, and set up tracking of future changes. In the last case, the `source` option must be specified as `ns`, `dir`, or `none` , and the selected side's content will overwrite the other side. **Use this with caution!**

## Current Limitations

* Link only handles correctly named namespaces - that is, observing `{⍵≡(⎕NS⍬)⍎⍕⍵}`. Scripted namespaces must simply be named. When creating an unscripted namespace, we recommend using `⎕NS` dyadically to name the created namespace (for example `'myproject'⎕NS⍬` rather than `myproject←⎕NS⍬`). This allows retrieving namespace reference from its display from (for example `#.myproject` rather than `#.[namespace]`).
* Link does not support namespace-tagged functions and operators (e.g. foo←namespace.{function}).
* Changes made using `←`, `⎕NS`, `⎕FX`, `⎕FIX`, `⎕CY`, `)NS` and `)COPY` or the APL line `∇` editor are not currently detected. For link to be aware of the change, they must be replaced by a call to [Link.Fix](/API/Link.Fix.md). Similarly, deletions with `⎕EX` or `)ERASE` must be replaced by a call to [Link.Expunge](/API/Link.Expunge.md).
* Link does not support files that define multiple names, whereas 2∘⎕FIX supports it.
* The detection of external changes to files and directories is currently only supported under .Net and .NetCore. However, changes to a file will be reflected in the namespace if the file's item is opened in APL's editor, on all platforms.
* Source code must not have embedded newlines within a string, whereas ⎕FX tolerates it. Link will error if this is attempted. This restriction comes because newline characters would be interpreted as a new line when saved as text file. When newlines characters must be used in source code, they should be implemented by a call to `⎕UCS` e.g. `newline←⎕UCS 13 10  ⍝ carriage-return + line-feed`
* Dyalog v18.1 is recommended if it is important that all source be preserved as typed. Earlier versions of APL have small glitches that occasionally lose the source as typed under certain circumstances. That includes handling :Require keywords, whitespaces in code, formatting of numeric constants, and more.
