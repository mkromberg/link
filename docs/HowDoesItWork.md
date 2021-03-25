## OK, but how does Link really work?

Some people need to know what is happening under the covers before they can relax and move on. If you are not one of those people, do not waste any further time on this section. If you do read it, understand that things may change under the covers without notice, and we will not allow a requirement to keep this document up-to-date to delay work on the code. It is reasonably accurate as of March 2021.

**Terminology:** In the following, the term *object* is used very loosely to refer to functions, operators, namespaces, classes and arrays.

### What Exactly is a Link?

A link connects a namespace in the active workspace (which can be the root directory, #) to a directory in the file system. When a link is created:

- An entry is created in the table held in `SE.Link.Links`, recording the endpoints and all options associated with the Link. The command `]Link.Status` can be used to report this information.
- APL Source files are created from workspace definitions, or objects are loaded into the workspace from source files, depending on which end of the link is specified as the source. These processes are described in more detail in the following.
- By default, a .NET File System Watcher is created to watch the directory for changes, so they can immediately be replicated in the workspace (if .NET is available)

#### Creating APL Source Files and Directories

Link writes textual representations of APL objects to UTF-8 text files. Most of the time, it uses the system function `⎕SRC` to extract the source form writing it to file using `⎕NPUT`. There are two exceptions to this:

* So-called "unscripted" namespaces which contain other objects but do not themselves have a textual source, are represented as sub-directories in the file system (which may contain source files for the objects within the namespace).
* Arrays are converted to source form using the function ` ⎕SE.Dyalog.Array.Serialise`. It is expected that `⎕SRC`will one day be extended to perform this function, but there is as yet no schedule for this.

#### Loading APL Objects from Source

As a general rule, Link loads code into the workspace using `2 ⎕FIX 'file://...'`. 

The interpreter keeps track of the links and updates source files when you "fix" code in the workspace. Once this is set up, editing objects will cause the editor itself (not Link) to update the source file. You can inspect existing links using a family of I-Beams numbered 517x. When a *new* function, operator, namespace or class is created, a hook in the editor calls Link code which generates a new file and sets up the link.

* If .NET is available, Link uses a File System Watcher to monitor linked directories and immediately react to file creation, modification or deletion.

From Link version 3.0, which was shipped with Dyalog version 18.1, an optional [Crawler](Crawler.md) will occasionally compare linked namespaces and directories, and deal with anything that might have been missed by the automatic mechanisms. This is especially useful if:

* The File System Watcher is not available on your platform
* You add functions or operators to the active workspace without using the editor, for example using ``)COPY`` or dfn assignment.

The document [Technical Details and Limitations](TechDetails.md) provides much more information about the type of APL objects that are supported by Link.

## Breaking Links

If [Link.Break](Link.Break.md) is used to explicitly break an existing Link, the entry is removed from `⎕SE.Link.Links`, and the namespace reverts to being a completely "normal" namespace in the workspace. If file system watch was active, the watcher is disabled. Any information that the interpreter was keeping about connections to files is removed using `5178⌶`. None of the definitions in the namespace are modified by the process of breaking a link.

If you delete a linked namespace using `)ERASE` or `⎕EX`, Link may not immediately detect that this has happened. However, if you call `Link.Status`, or make a change to a watch file that causes the file system watched to attempt to update the namespace, Link will discover that something is amiss, issue a warning, and delete the link.

If you completely destroy the active workspace using `)LOAD` or `)CLEAR`, all links will be deleted.

**