---
{"publish":true,"title":"chgrp","created":"2025-08-29","modified":"2025-08-30T10:35:59.033-04:00","published":"2025-08-29","tags":["commands"],"cssclasses":""}
---

## About chgrp

Changes group ownership of a file or files.

### Description

Change the group of each **FILE** to **GROUP**. With **--reference**, change the group of each **FILE** to that of **RFILE**.

## chgrp syntax

    chgrp [OPTION]... GROUP FILE...

    chgrp [OPTION]... --reference=RFILE FILE...

### Options

|   |   |
| :-- | :-- |
| -c, --changes | Like verbose but report only when a change is made. |
| -f, --silent, --quiet | Suppress most error messages. |
| -v, --verbose | Output a diagnostic for every file processed. |
| --dereference | Affect the referenced file of each symbolic link, rather than the symbolic link itself, which is default setting. |
| -h, --no-dereference | Affect symbolic links instead of any referenced file. This option is useful only on systems that can change the ownership of a symlink. |
| --no-preserve-root | Do not treat '/' in any special way. This option is the default. |
| --preserve-root | Do not operate recursively on '/'. |
| --reference=RFILE | Use RFILE's group rather than specifying a GROUP value. |
| -R, --recursive | Operate on files and directories recursively. |

The following options modify how a hierarchy is traversed when the **-R** option is also specified. If more than one of these options is specified, only the final one takes effect:

|   |   |
| :-- | :-- |
| -H | If a command line argument is a symbolic link to a directory, traverse it. |
| -L | Traverse every symbolic link to a directory. |
| -P | Do not traverse any symbolic links. This option is the default. |
| --help | Display a help message and exit. |
| --version | Output version information and exit. |

## chgrp examples

    chgrp hope file.txt

Change the owning group of the file **file.txt** to the group named **hope**.

    chgrp -hR staff /office/files

Change the owning group of **/office/files**, and all subdirectories, to the group **staff**.

## Related commands

* [chmod](../chmod) — Change the permissions of files or directories.
* [chown](../chown) — Change the ownership of files or directories.
* id — Display real and effective user and group IDs.
