### Setting up your Environment

With a small project, you can get by using `]Link.Create` and/or `Link.Import` to bring your source into the workspace in order to work with it. However, even in a small project, this quickly gets tedious, and as the project grows, you may want to load code from more that one directory, and perhaps run some code in order to set things up or even start the application. Fortunately, the [Link API](API.md) provides all the functions that you need to automate the setup.

To illustrate, we will create a small application that uses the stats library that we created in the [introduction](GettingStarted.md). We'll put the application into a namespace called `linkdemo`:

```      apl
      )clear
clear ws
      )ns linkdemo
      ]link.create linkdemo /users/sally/linkdemo
Linked: #.linkdemo ←→ /users/sally/linkdemo

      )ed linkdemo.Run
```

Our application is going to prompt the user for an input array and output the mean and standard deviation of the data, until the user inputs an empty array. Obviously, the code should be enhanced to validate the input and perhaps trap errors, but that is left as an exercise for the reader.

```apl
     ∇ Main;data                                           
[1]   ⍝ Compute Mean and StdDev until user inputs an empty array
[2]                                                             
[3]    :Repeat                                                  
[4]        ⎕←'Enter some numbers:'                              
[5]        :If 0≠⍴data←⎕                                        
[6]            ⎕←'Mean:   ',1⍕#.stats.Mean data                      
[7]            ⎕←'StdDev: ',1⍕#.stats.StdDev data                    
[8]        :EndIf                                               
[9]    :Until 0=≢data                                           
     ∇                                                                                   
```

We will need the `stats` code in the workspace as well, of course. Since we only intend to use it and don't want to risk making changes to its source code while testing our own application, we will use `]link.import` rather than `]link.create` to bring that code into the workspace:

```      apl
      ]link.import stats /users/sally/stats
Imported: #.starts ← c:\tmp\stats

      linkdemo.Main
      Enter some numbers:
⎕:
      50+?100⍴100
Mean:    102.4
StdDev:  30.1
Enter some numbers:
⎕:
      ⍬
```

### Automating Startup

Starting with version 18.0, it is simple to launch the interpreter from a text file: either a source file defining a function, namespace or class using the [LOAD parameter](https://help.dyalog.com/18.0/#UserGuide/Installation%20and%20Configuration/Configuration%20Parameters/Load.htm) or from a configuration file using the  [CONFIGFILE parameter](https://help.dyalog.com/18.0/#UserGuide/Installation%20and%20Configuration/Configuration%20Files.htm). Configuration files allow you to both set a startup expression and include other configuration options for the interpreter. For example, if we were to define a file `dev.dcfg` in the `linkdemo` folder with the following contents:

```json
{
  Settings: {
      MAXWS: 100M,
      LX: "linkdemo.Start 0 ⊣ ⎕←⎕SE.Link.Create 'linkdemo' ⎕SE.Link.LaunchDir"
  }
}
```

This specifies an APL session with a MAXWS of 100 megabytes, which will start by creating the `linkdemo`  namespace and calling `linkdemo.Start`. The namespace will be created using the directory named by the result of the function `⎕SE.Link.LaunchDir`; this will be the directory that the CONFIGFILE parameter refers to (or, if there is no CONFIGFILE, the directory referred to by the LOAD parameter).

The function `linkdemo.Start` will bring in the `stats` library using `Link.Import`: since we are not developers of this library, we don't want to create a bi-directional link that might allow us to accidentally modify it during our testing. It also creates the name `ST` to point to the stats library, which means that our `Run` function can use more pleasant names, like `ST.Mean` in place of `#.stats.Mean` - which also makes it easier to relocate that module in the workspace:

```apl
     ∇ Start run                                                                       
[1]   ⍝ Establish development environment for the linkdemo application                 
[2]                                                                                    
[3]    ⎕IO←⎕ML←1                                                                       
[4]    ⎕SE.Link.Import '#.stats' '/home/sally/stats' ⍝ Load the stats library
[5]    ST←#.stats                                                                      
[6]
[7]    :If run                                                                         
[8]        Main                                                                        
[9]        ⎕OFF                                                                        
[10]   :EndIf                                                                          
     ∇                                                                                 
```

We can now launch out development environment using `dyalog CONFIGFILE=linkdemo/devt.cfg`, or on some platforms right-clicking on this file and selecting Run.

### Development vs Runtime

The `Start`function takes a right argument `run` which decides whether it should just exit after initialising the environment, or it should launch the application by calling `Run`and terminate the session when the user decides that the job is done.

This allows us to create a second configuration file, `linkdemo/run.dcfg`, which differs from `dev.dcfg` in that we reserve a bigger workspace (since we'll be doing real work rather than just testing), and brings the source code in using `Link.Import` rather than `Link.Create`, which means that we won't waste resources setting up a file system watcher, and that accidental changes made by anyone running the application will not update the source files.

```json
{
  Settings: {
      MAXWS: 1G,
      LX: "linkdemo.Start 1 ⊣ ⎕←⎕SE.Link.Import 'linkdemo' ⎕SE.Link.LaunchDir"
  }
}
```

### Distribution Workspace

As we have seen, Link allows you to run your application based entirely on textual source files. However, if you have a lot of source files it may be more convenient for the users of your application to receive a single workspace file with all of the source loaded. 

To prepare a workspace for shipment, we will need to:

* Set `⎕LX` in the so that it calls the `Start` function
* Use [Link.Break](Link.Break.md) to remove links to the source files. If you omit this step, you can create a [potentially confusing situation](Workspaces.md#Saving-a-Workspace-with-Links).
* `)SAVE` the workspace