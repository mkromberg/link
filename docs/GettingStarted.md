# Getting started with `Link`

### Linking to an Existing Directory

In daily use, the assumption is that your source already exists in one or more file system folders that you are managing using an SCM like Git, or taking copies of at suitable intervals using some other mechanism. You will use [Link.Create](Link.Create.md) to load the source code into the workspace so that you can work with it. For example, the following user command load all the code in the folder `/home/sally/myapp` into a namespace called `myapp` , creating the namespace if it does not exist: 

```      apl
      ]link.Create myapp /home/sally/myapp
```

The first argument to the user command, and to the corresponding API function `⎕SE.Link.Create` is the name of a namespace (in the case of the function you could also use a reference to an existing namespace). The second argument is a directory name.

Of course, you might not already have a source directory, so let's look at how you might create one.

### Starting a New project

If you are starting a brand new project, you can use the same command as above, but you will need to create either the namespace or the folder (or both) first. For the command to succeed, at least one of them needs to exist and exactly one of them needs to contain some code.

* If neither of them exist, Link.Create will reject the request on suspicion that there is a typo, in order to avoid silently creating an empty directory by mistake.
* If both of them exist AND contain code, and the code is not identical on both sides, Link.Create will fail and you will need to specify the  `source` option, whether the namespace or the directory should be considered to be the source. Incorrectly specifying the source will potentially overwriting any content on the other side, so use this with extreme caution!

To illustrate, we will create a namespace and populate it with two dfns and one tradfn, in order to have something to work with. In this example, the functions are created under program control; under normal use the functions would probably be created using the editor:

```apl
      'stats' ⎕NS ⍬ ⍝ Create an empty namespace
      stats.⎕FX 'mean←Mean vals;sum' 'sum←+⌿,vals' 'mean←sum÷1⌈⍴,vals'
      stats.Root←{⍺←2 ⋄ ⍵*÷⍺}
      stats.StdDev←{2 Root(+.×⍨÷⍴),⍵-Mean ⍵}
```
We could now create a source directory using [Link.Export](Link.Export.md), and then use [Link.Create](Link.Create.md) to create a link to it. However, [Link.Create](Link.Create.md) can do this in one step: assuming that the directory `/tmp/stats` is empty or does not exist, the following command will detect that there is code in the workspace but not in the directory, and create a link based on the namespace that we just created:

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

If you have an existing workspace containing several namespaces, code in the root of the workspace, or variables, you will want to read about [converting your workspace to text source](ExportingSource.md).

### Working with the Code

Once the link is set up, you can work with your code using the Dyalog IDE exactly as you would if you were not using Link; the only difference being that Link will ensure that any changes you make to the code within the `stats`namespace are instantly copied to the corresponding source file.

Conversely, if you are new to Dyalog APL, and have a favourite editor, you can use it to edit the source files directly, and any change that you make will be replicated in the active workspace. If you do not have a File System Watcher available on your platform, it may be a few seconds before the [Crawler](Crawler.md) kicks in and detects external changes.

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

When changes are made using the editor which is built-in to Dyalog IDE or RIDE, source files are updated immediately. Changes made outside the editor will not immediately be picked up. This includes:

* Source code created or changed using assignment (`←`), `⎕FX`  or `⎕FIX` - or the APL line "`∇`" editor.
* Definitions moved between workspaces or namespaces using `⎕CY`, `⎕NS` or `)COPY`.
* Definitions erased using `⎕EX`or `)ERASE`

By default, a configurable [Crawler](Crawler.md) will wake up ever few seconds and scan your workspace and the source directory for any changes that were not detected.

If you write tools which modify source code under program control, it is a good idea to call the API functions [Link.Fix](Link.Fix.md) or [Link.Expunge](Link.Expunge.md) to inform link that you have made the change, without relying on the crawler being active. The crawler is configurable and may have been disabled by the user.

If you update the source files under program control and inbound synchronisation is not enabled, you can use [Link.Notify](Link.Notify.md) to let Link know about an external change that you would like to bring into the workspace.

### Arrays

By default, Link does not consider arrays to be part of the source code of an application and will not write arrays to source files unless you explicitly request it. Link is not intended to be used as a database management system; if you have arrays that are modified during the normal running of your application, we recommend that you store that data in an RDBMS or other files that are managed by the application code, rather than using Link for this.

If you have arrays that represent error tables, range definitions or other *constant* inputs to the application, that should be considered to be part of the source code, you can cause them to be written to file using [Link.Add](Link.Add.md):

```apl
      stats.Directions←'North' 'South' 'East' 'West'
      ]Link.Add stats.Directions
Added: #.stats.Directions
```

Note that by default, source files for arrays are presumed to define the initial value of the array when the application starts. By default, changes made to arrays will not be picked up by a crawler even if the array was originally loaded from a source file.  You always need to explicitly call Link.Add to update the source file with a new value. 

Although changes to the array in the workspace are not automatically written to file, changes made to the source files will immediately be reflected in the workspace.

### Setting up Development and Runtime Environments

We have seen how to use `]Link.Create`to load textual source into the workspace in order to work with it. As your project grows, you will probably want to split your code into modules, for example application code in one directory and shared utilities in another - and maybe also run some code to get things set up.

Next, we will look at [Setting up Development and Runtime Environments](Setup.md), so that you don't have to type the same sequence of things over and over again to get started with development - or running the application.

