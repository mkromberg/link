# Installation

The fully supported version of Link 3.0 is included with Dyalog version 18.1 or later; no installation is required. The instructions on this page only apply if you want to:

* Use Link 3.0 in place of Link 2.0 with Dyalog version 18.0
* Participate in testing pre-releases of Link
* Have some other reason for wanting to use a different version than that which is distributed with APL.

Link is maintained as an open source project at [https://github.com/dyalog/link](https://github.com/dyalog/link). Once you have downloaded or cloned a particular tag or release of the source code to a folder on your machine, you have two alternatives:

* **OVERWRITE the installed Link** by replacing the contents of the **$DYALOG/StartupSession** in your installation with the contents of the StartupSession folder that you have downloaded.
* **Set the `DYALOGSTARTUPSE` environment variable** to point to the StartupSession folder that you have downloaded. This route is recommended if you think you will regularly be updating your copy of Link; if you have done a `git clone` of the repository, then all you need to do is a `git pull` and restart APL.

If you are using Dyalog APL version 18.0, you also need to update the user command file used to invoke Links user commands. This should only need to be done once, even if you subsequently download new versions of Link. Again, there are two options:

* Overwrite the installed file **$DYALOG/SALT/spice/Link.dyalog** with the corresponding file that you have downloaded.
* Place a copy of the downloaded file into your `MyUcmds` folder, which will cause it to take priority over the installed copy.

You will need to restart Dyalog APL each time you update these files, in order to pick up the new definitions.