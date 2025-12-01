---
title: "Re-mounting a drive for data syncing"
author: Penelope How
date: 2023-01-09 13:00
classes: wide
categories:
  - Guides
tags: 
  - aws
---
## How to re-mount the a remote drive

If the drive is unmounted, there are some simple steps that you can perform to re-mount the drive and fetch the latest data. First, connect to the server/mounting point where the workflow is being performed. It is likely that you will be prompted for your username and password. Please remember that these are credentials for access to the server, not your local credentials.

Next, we can remount the  drive as so:

```
$ sudo mount -rw -t cifs -o username=<USERNAME>,uid=<UID>,gid=<GID> <REMOTE-PATH> <LOCAL-PATH>
```

Where `<USERNAME>` is the username for access to the external mount, and `<UID>` and `<GID>` are the identifiers associated with your username and your group. If you do not know the `<UID>` and `<GID>` then you can look them up with this command:

```
$ id
```

Once the  drive is remounted, perform the following `rsync` routine to fetch the latest data:

```
$ sudo rsync -r --exclude .git <SOURCE-PATH> <TARGET-PATH>
```

Be aware that you may need sudo privileges in order to do this. You may be prompted for two passwords - one for your target drive user (if mounting to an external drive) and another for your source account.

If no new data appears on the target drive after performing these steps then it is likely that the problems lies in the workflow.
