# Getting Started

### Linking to an Existing Directory

In daily use, the assumption is that your source already exists in one or more file system folders that you are managing using an SCM like Git, or taking copies of at suitable intervals using some other mechanism. [Link.Create](/API/Link.Create.md) loads the source code into the workspace so that you can work with it, keeping the content of the workspace synchronised with the source files, no matter which side of the link you make changes.

For example, the following user command load all the code in the folder `/home/sally/myapp` into a namespace called `myapp` , creating the namespace if it does not exist: 

```      apl
      ]link.Create myapp /home/sally/myapp
```

The first argument to the user command, and to the corresponding API function `⎕SE.Link.Create` is the name of a namespace (in the case of the function you could also use a reference to an existing namespace). The second argument is a directory name.

### Starting with a Workspace

If your existing code is in a workspace rather than in source files, you may want to skip forward and read the section on [converting a workspace to source files](WStoLink.md) before continuing.

### Starting a New project

Of course, you might be one of those lucky people who is starting a completely new project. In this casem you can still use `Link.Create`, but you will need to create either the namespace or the folder (or both) first. For the command to succeed, at least one of them needs to exist and exactly one of them needs to contain some code.

* If neither of them exist, Link.Create will reject the request on suspicion that there is a typo, in order to avoid silently creating an empty directory by mistake.
* If both of them exist AND contain code, and the code is not identical on both sides, Link.Create will fail and you will need to specify the  `source` option, whether the namespace or the directory should be considered to be the source. Incorrectly specifying the source will potentially overwrite existing content on the other side, so use this with extreme caution!

To illustrate, we will create a namespace and populate it with two dfns and one tradfn, in order to have something to work with. In this example, the functions are created using APL expressions; under normal use the functions would probably be created using the editor, or perhaps loaded or copied from an existing workspace.

```apl
      'stats' ⎕NS ⍬ ⍝ Create an empty namespace
      stats.⎕FX 'mean←Mean vals;sum' 'sum←+⌿,vals' 'mean←sum÷1⌈⍴,vals'
      stats.Root←{⍺←2 ⋄ ⍵*÷⍺}
      stats.StdDev←{2 Root(+.×⍨÷⍴),⍵-Mean ⍵}
```
We could now create a source directory using [Link.Export](/API/Link.Export.md), and then use [Link.Create](/API/Link.Create.md) to create a link to it. However, [Link.Create](/API/Link.Create.md) can do this in one step: assuming that the directory `/tmp/stats` is empty or does not exist, the following command will detect that there is code in the workspace but not in the directory, and create a link based on the namespace that we just created:

```apl
      ]LINK.Create stats /tmp/stats -source=ns
Linked: #.stats ←→ C:\tmp\stats
```
The double arrow `←→` in the output indicates that synchronisation is bi-directional. We can verify that the three expected files have been created:

```apl
      ls←⎕NINFO⍠1 ⍝ List files, allowing wildcards
      ls '/tmp/stats/*'
  /tmp/stats/Mean.aplf  /tmp/stats/Root.aplf  /tmp/stats/StdDev.aplf  
```
Let's verify that our source directory can be used to re-build the original namespace::

```apl
      )CLEAR
clear ws
      ]LINK.Create stats /tmp/stats
Linked: stats ←→ C:\tmp\stats
      stats.⎕NL -3 ⍝ Verify functions were loaded as expected
 Mean  Root  StdDev
```

If you have an existing workspace containing several namespaces, code in the root of the workspace, or variables, you will want to read about [converting your workspace to text source](WStoLink.md).

### Working with the Code

Once the link is set up, you can work with your code using the Dyalog IDE exactly as you would if you were not using Link; the only difference being that Link will ensure that any changes you make to the code within the `stats`namespace are instantly copied to the corresponding source file. 

NB: In the context of this document, the term *Dyalog IDE* includes both the Windows IDE and the Remote IDE (RIDE), which is tightly integrated with the interpreter.

Conversely, if you are new to Dyalog APL, and have a favourite editor, you can use it to edit the source files directly, and any change that you make will be replicated in the active workspace. If you do not have a File System Watcher available on your platform, it may be a few seconds before the [Crawler](/Crawler.md) kicks in and detects external changes.

If you use editors inside or outside the APL system to add new functions, operators, namespaces or classes,  the corresponding change will be made on the other side of the link. For example, we could add a `Median` function:

```apl
      )ED stats.Median
```

In the Edit window, we complete the function:

```apl
Median←{
     asc←⍋vals←,⍵
     Mean vals[asc[⌈2÷⍨0 1+⍴vals]]
 }
```

When the editor fixes the definition of the function in the workspace, Link will create a new file:


```apl
      ls '/tmp/stats/*'
  /tmp/stats/Mean.aplf  /tmp/stats/Median.aplf  /tmp/stats/Root.aplf  /tmp/stats/StdDev.aplf  
```

### Changes made Outside the Editor

When changes are made using the editor which is built-in to Dyalog IDE (which includes RIDE), source files are updated immediately. Changes made outside the editor will not immediately be picked up. This includes:

* Definitions created or changed using assignment (`←`), `⎕FX`  or `⎕FIX` - or the APL line "`∇`" editor.
* Definitions moved between workspaces or namespaces using `⎕CY`, `⎕NS` or `)COPY`.
* Definitions erased using `⎕EX`or `)ERASE`

If you write tools which modify source code under program control, it is a good idea to call the API functions [Link.Fix](/API/Link.Fix.md) or [Link.Expunge](/API/Link.Expunge.md) to inform Link that you have made the change.

If you update the source files under program control and inbound synchronisation is not enabled, you can use [Link.Notify](/API/Link.Notify.md) to let Link know about an external change that you would like to bring into the workspace.

### Arrays

By default, Link does not consider arrays to be part of the source code of an application and will not write arrays to source files unless you explicitly request it. Link is not intended to be used as a database management system; if you have arrays that are modified during the normal running of your application, we recommend that you store that data in an RDBMS or other files that are managed by the application code, rather than using Link for this.

If you have arrays that represent error tables, range definitions or other *constant* definitions that it makes sense to conside to be part of the source code, you can add them using [Link.Add](/API/Link.Add.md):

```apl
      stats.Directions←'North' 'South' 'East' 'West'
      ]Link.Add stats.Directions
Added: #.stats.Directions
```

Note that by default, source files for arrays are presumed to define the initial value of the array when the application starts. Note that changes made to arrays will not be picked up by a crawler even if the array was originally loaded from a source file.  You always need to use the built-in editor or explicitly call Link.Add to update the source file with a new value.

Although changes to the array in the workspace are not automatically written to file, changes made to the source files will always be considered to be updates to the source code and reflected in the workspace if synchronisation is active.

### Setting up Development and Runtime Environments

We have seen how to use `]Link.Create`to load textual source into the workspace in order to work with it. As your project grows, you will probably want to split your code into modules, for example application code in one directory and shared utilities in another - and maybe also run some code to get things set up.

Next, we will look at [Setting up Development and Runtime Environments](Setup.md), so that you don't have to type the same sequence of things over and over again to get started with development - or running the application.
