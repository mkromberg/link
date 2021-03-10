### OK, but how does Link really work?

Some people need to know what is happening under the covers before they can relax and move on. If you are not one of those people, do not waste any further time on this section. If you do read it, understand that things may change under the covers without notice, and we will not allow a requirement to keep this document up-to-date to delay work on the code. It is reasonably accurate as of March 3rd, 2021.

**Terminology:** In the following, the term *object* is used very loosely to refer to functions, operators, namespaces, classes and arrays.

#### Creating APL Source Files and Directories

Link writes textual representations of APL objects to UTF-8 text files. Most of the time, it uses the system function `⎕SRC` to extract the source form writing it to file using `⎕NPUT`. There are two exceptions to this:

* So-called "unscripted" namespaces which contain other objects but do not themselves have a textual source, are represented as sub-directories in the file system (which may contain source files for the objects within the namespace).
* Arrays are converted to source form using the function ` ⎕SE.Dyalog.Array.Serialise`. It is expected that `⎕SRC`will one day be extended to perform this function, but there is as yet no schedule for this.

#### Loading APL Objects from Source

As a general rule, Link loads code into the workspace using `2 ⎕FIX 'file://...'`. 

The interpreter keeps track of the links and updates source files when you "fix" code in the workspace. You can inspect existing links using a family of I-Beams numbered 517x. When a new function or operator is created, a hook in the editor calls some Link code which generates a new file and sets up the link.

* If .NET is available, Link uses a File System Watcher to monitor linked directories and immediately react to file creation, modification or deletion.

From Link version 3.0, which was shipped with Dyalog version 18.1, an optional ***crawler*** will occasionally compare linked namespaces and directories, and deal with anything that might have been missed by the automatic mechanisms. This is especially useful if:

* The File System Watcher is not available on your platform
* You add functions or operators to the active workspace without using the editor, for example using ``)COPY`` or dfn assignment.

The document [Technical Details and Limitations](TechDetails.md) provides much more information about the type of APL objects that are supported by Link.