---
title: "Clion Miscellaneous"
tags: ["clion"]
---
Clion Miscellaneous Things:

# clion cannot find declaration to go to

Help menu > Edit Custom Properties
```shell
# Maximum file size (kilobytes)
idea.max.intellisense.filesize=2500
```

# clion environment variables

If you use snap to install clion, then the `clion_clion.desktop` file can be found in `/var/lib/snapd/desktop/applications`. Copy this file into some folder and right click on the file to `Allow Launching`.
Edit the field `Exec` as follows:

```
Exec=/bin/sh /snap/clion/114/bin/clion.sh
```
In such way, the clion will be started with all environment variables inherited.
<!--more-->
