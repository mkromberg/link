# Installation

A fully supported version Link is included with Dyalog version 18.0 or later; no installation is required. 

## Installing from GitHub ##

Link is maintained as an open source project at [https://github.com/dyalog/link](https://github.com/dyalog/link), which means that you have the option of using a different version from that which is distributed with APL, for example if you want to participate in testing pre-releases of Link.

Once you have downloaded or cloned the source code to your local machine, you have two choices:

* Either copy the contents of the checked out link repository into the Dyalog installation directory in such a way that  **startup.dyalog** ends up in the same directory as **dyalog.exe**, confirming the overwriting of **$DYALOG/SALT/spice/Link.dyalog** and of **$DYALOG/StartupSession/\***.
* Or set the `DYALOGSTARTUPSE` environment variable to point to the StartupSession folder in the checkout.

You will need to restart Dyalog APL.