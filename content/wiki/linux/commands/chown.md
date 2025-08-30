---
{"publish":true,"title":"chown","created":"2025-08-29","modified":"2025-08-30T10:48:10.301-04:00","published":"2025-08-29","tags":["commands"],"cssclasses":""}
---

## About chown

The **chown** command changes the owner and owning group of files.

### Description

**chown** changes the user or group ownership of each given file. If only an owner (a user name or numeric user ID) is given, that user is made the owner of each given file, and the files' group is not changed. If the owner is followed by a colon and a group name (or numeric group ID), with no spaces between them, the group ownership of the files is changed as well. If a colon but no group name follows the user name, that user is made the owner of the files and the group of the files is changed to that user's login group. If the colon and group are given, but the owner is omitted, only the group of the files is changed; in this case, **chown** performs the same function as chgrp. If only a colon is given, or if the entire operand is empty, neither the owner nor the group is changed.

## chown syntax

    chown [OPTION]... [OWNER][:[GROUP]] FILE...
    chown [OPTION]... --reference=RFILE FILE... 

### Options

Change the owner or group of each **FILE** to **OWNER** or **GROUP**. With **--reference**, change the owner and group of each **FILE** to those of **RFILE**.

|  |  |
| :-- | :-- |
| -c, --changes | Like verbose but report only when a change is made. |
| -f, --silent, --quiet | Suppress most error messages. |
| -v, --verbose | Output a diagnostic for every file processed. |
| --dereference | Affect the referenced file of each symbolic link rather than the symbolic link itself, which is the default setting. |
| -h, --no-dereference | Affect symbolic links instead of any referenced file. This option is useful only on systems that can change the ownership of a symlink. |
| --from=CURRENT_OWNER:CURRENT_GROUP | Change the owner or group of each file only if its current owner or group match those specified here. Either may be omitted, in which case a match is not required for the omitted attribute. |
| --no-preserve-root | Do not treat '/' (the root directory) in any special way, which is the default setting. |
| --preserve-root | Do not operate recursively on '/'. |
| --reference=RFILE | Use RFILE's owner and group rather than specifying OWNER:GROUP values. |
| -R, --recursive | Operate on files and directories recursively. |

The following options modify how a hierarchy is traversed when the **-R** option is also specified. If more than one is specified, only the final one takes effect.

|  |  |
| :-- | :-- |
| -H | If a command line argument is a symbolic link to a directory, traverse it. |
| -L | Traverse every symbolic link to a directory encountered |
| -P | Do not traverse any symbolic links, which is the default setting. |
| --help | Display this help and exit. |
| --version | Output version information and exit. |

Owner is unchanged if unspecified. Group is unchanged if unspecified, or changed to the login group if implied by a '**:**' following a symbolic **OWNER**. **OWNER** and **GROUP** may be numeric as well as symbolic.

## chown examples

    chown hope file.txt

Set the owner of file **file.txt** to user **hope**.

    chown -R hope /files/work

Recursively grant ownership of the directory **/files/work**, and all files and subdirectories, to user **hope**.

## Related commands

* [chgrp](../chgrp) — Change the group ownership of files or directories.
* [chmod](../chmod) — Change the permissions of files or directories.
* [ls](../ls) — List the contents of a directory or directories.
