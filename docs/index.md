# Introduction

This is an introduction to Link for users of Dyalog APL installations that include Link 3.0 (version 18.0 or later). If you have an earlier version of APL or Link, you may want to check out one or more of the following pages before continuing: 

* [Migrating to Link 3.0 from from Link 2.0](Upgradeto30.md)
* [Migrating to Link 3.0 from SALT](GettingStarted/SALTtoLink.md)
* [Installation instructions:](GettingStarted/Installation.md) If you want to pick Link up directly from the GitHub repository rather than use the version installed with APL.
* [How does Link work?](Discussion/HowDoesItWork.md) If you are one of those people who cannot comfortably use a tool until they understand the technical details of how it works, start here.
* [The historical perspective:](Discussion/History.md) Link is a step on a journey which begins more than a decade ago with the introduction of SALT for managing source code in text files, as an alternative to binary workspaces and files, and will hopefully end with the interpreter handling everything itself.

## What is Link?

***Link*** supports Unicode text files as the recommended way to store APL source code, rather than the traditional binary workspace. The benefits of using Link and text files include:

* It is easy to use source code management tools like Git or Subversion to manage your code.

  *Note that an SCM is not a requirement for using Link*; without an SCM you just need your own strategy for taking suitable copies of your source files, as your would with workspaces.

* Changes to your code are **immediately** written to file: there is no need to remember to save your work. You normally make the record permanent with a *commit* to your source code management system.
  
* Unlike binary workspaces, text source can be shared between different versions of APL - or even with human readers or writers don't have APL installed at all.

* Source code stored in external files is preserved exactly as typed, rather than being reconstructed from the tokenised form.

Although an SCM is not a requirement for Link: if you are not already using a source code management system, we ***highly*** recommend making the effort to [learn about - and install Git](GettingStarted/SCMforAPLers.md).

## Link is NOT...

* **A source code management system**: we recommend using Git to manage the text files that Link will help you create and edit using Dyalog APL.
* **A database management system:** although Link is able to store APL arrays using a pre-release of the *literal array notation*, this is only intended to be used for constants which you consider to be part of the source code of your applications. Although all functions and operators that you define will be written to source files by default, arrays are only written to source files upon request using [Link.Add](API/Link.Add.md) or by specifying optional parameters to [Link.Export](API/Link.Export.md). Application data should be stored in a database management system or files managed by the application.

## Link Fundamentals

Link establishes ***links*** between one or more **namespaces** in the active APL workspace and corresponding **directories** containing APL source code in Unicode test files. For example, the following user command invocation will link a namespace called `myapp` to the folder `/home/sally/myapp`:

```      apl
      ]link.Create myapp /home/sally/myapp
```

If `myapp` contains sub-directories, a namespace hierarchy corresponding to the directory structure will be created within the `myapp` namespace. By default, the link is bi-directional, which means that Link will:

* **Keep Source Files up-to-date:** 
Any changes made to code in the active workspace using the tracer and editor are immediately replicated in the corresponding text files.
* **Keep the Workspace up-to-date:**
Any changes made to the external files using a text editor, or resulting from an SCM action such as rolling back or switching to different branch, will immediately be reflected in the active workspace.

You can invoke `]Link.Create`several times to create multiple links, and you can also use `]Link.Import`or `]Link.Export` to import source code into the workspace or export code to external files *without* creating links that will respond to subsequent changes. It is also possible to set links up that only react to changes at one end of the link.

## Link Functions vs. User Commands

For every user command like `]Link.Create`, there is a corresponding function with a name like `⎕SE.Link.Create`.  The user commands are intended for experimentation or simple applications which do consist of a single directory and are only used interactively.

If there are several steps involved in creating your application runtime environment, or you need to deploy your application in such a way that it starts up automatically, you will be better off using the functions and calling them from a function or script that you use to perform initialisation.

The details of the arguments to the functions and the user commands can be found in the [API Reference](API/index.md).

## Further Reading

To continue your journey towards getting set up with Link, you will want to read:

* [Setting up your environment](GettingStarted/Setup.md), for a discussion of how to set up Link-based development and runtime environments.
* [Technical Details and Limitations](Discussion/TechDetails.md), if you want to know about the full range of APL objects that are supported, and some of the edge cases that are not yet supported by Link.

If you have an existing APL application that you want to move to Link, you may want to read one of the following texts first:

* [Converting your workspace to text source](GettingStarted/WStoLink.md) (if you already have an existing body of APL code that is not in Link, you may want to read).
* [Migrating to Link 3.0 from SALT](GettingStarted/SALTtoLink.md), if you are already managing text source using Link's predecessor SALT.

## Frequently Asked Questions

* [How does Link work?](Discussion/HowDoesItWork.md)
* [What happens if I save a workspace after creating Links?](Discussion/Workspaces.md)
* [Are workspaces dead now?](Discussion/Workspaces.md)
