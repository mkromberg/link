# The Crawler

In a future version of Link, an optional *Crawler* will occasionally perform a [Link.Resync](API/Link.Resync.md) in the background, to detect new code objects added to the workspace using other mechanisms than the editor, or source file changes that were not picked up by the file system watcher. In environments whether file system watching is not possible, the crawler can replace the FSW.

We hope to make the crawler available in the next release of Link, version 3.1 - during 2021.