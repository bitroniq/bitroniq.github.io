Below are the flags (chrome://flags) I have changed in Chrome 69 64-bit (September 2018) to [indicated values] to speed up the startup experience with many open tabs and windows.

The memory consumption is still high (1.2 GB in my case) but Chrome starts instantly and doesn't seem to be busy reloading all of the tabs.

Only Auto-Reload Visible Tabs [Enabled]
Pages that fail to load while the browser is offline will only be auto-reloaded if their tab is visible. – Mac, Windows, Linux, Chrome OS, Android

#enable-offline-auto-reload-visible-only

Automatic tab discarding [Enabled]
If enabled, tabs get automatically discarded from memory when the system memory is low. Discarded tabs are still visible on the tab strip and get reloaded when clicked on. Info about discarded tabs can be found at chrome://discards. – Mac, Windows

#automatic-tab-discarding

Infinite Session Restore [Enabled]
Reduces the number of tabs being loaded simultaneously during session restore, to improve responsiveness of the foreground tab. This requires #enable-page-almost-idle. – Mac, Windows, Linux, Chrome OS

#infinite-session-restore

Page Almost Idle [Enabled]
Make session restore use a definition of loading that waits for CPU and network quiescence. – Mac, Windows, Linux, Chrome OS

#page-almost-idle

Proactive Tab Freeze and Discard [Enabled Freeze and Discard]
Enables proactive tab freezing and discarding. This requires #enable-page-almost-idle. – Mac, Windows, Linux, Chrome OS

#proactive-tab-freeze-and-discard

Site Characteristics database [Enabled] (this one is probably not necessary)
Records usage of some features in a database while a tab is in background (title/favicon update, audio playback or usage of non-persistent notifications). – Mac, Windows, Linux, Chrome OS

#site-characteristics-database


--------------------
My settings:
chrome://discards/
chrome://flags/#expensive-background-timer-throttling
chrome://flags/#automatic-tab-discarding
chrome://flags/#proactive-tab-freeze-and-discard (enable freeze only)
chrome://flags/#page-almost-idle
chrome://flags/#infinite-session-restore
chrome://flags/#site-characteristics-database
chrome://flags/#enable-offline-auto-reload-visible-only



