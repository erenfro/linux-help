---
{"publish":true,"title":"ls","created":"2025-08-29","modified":"2025-08-30T10:49:22.396-04:00","published":"2025-08-29","tags":["commands"],"cssclasses":""}
---

# About ls

Lists the contents of a directory.

# Syntax

    ls [OPTION]... [FILE]...

# Description

List information about the FILEs (the current directory by default). Sort entries alphabetically if none of **-cftuvSUX** nor **--sort** is specified.

|   |   |
| :-- | :-- |
| -a, --all | Do not ignore entries starting with ".", providing visibility to hidden files (those starting with a ".") |
| -A, --almost-all | Do not list implied "." and "..". |
| --author | With -l, print the author of each file. |
| -b, --escape | Print C-style escapes for nongraphic characters. |
| --block-size=SIZE | Scale sizes by SIZE before printing them. E.g., '--block-size=M' prints sizes in units of 1,048,576 bytes. See SIZE format below. |
| -B, --ignore-backups | Do not list implied entries ending with "~". |
| -c | With -lt:, sort by and show the ctime (time of last modification of file status information); with -l:, show ctime and sort by name; otherwise: sort by ctime, newest first. |
| -C | List entries by columns. |
| --color[=WHEN] | Colorize the output. WHEN defaults to 'always' or can be 'never' or 'auto'. |
| -d, --directory | List directory entries instead of contents, and do not dereference symbolic links. |
| -D, --dired | Generate output designed for Emacs' dired mode. |
| -f | Do not sort, enable -aU, and disable -ls --color. |
| -F, --classify | Append indicator (one of */=>@|) to entries. |
| --file-type | Similar to --classify, except do not append '*' |
| --format=WORD | Formats according to the following: across -x, commas -m, horizontal -x, long -l, single-column -1, verbose -l, vertical -C. |
| --full-time | Like -l --time-style=full-iso. |
| -g | Like -l, but do not list owner. |
| --group-directories-first | Group directories before files. Can be augmented with a --sort option, but any use of --sort=none (-U) disables grouping. |
| -G, --no-group | In a long listing, don't print group names. |
| -h, --human-readable | With -l, print sizes in human-readable format (e.g., 1K, 234M, 2G). |
| --si | Like --human-readable, but use powers of 1000, not 1024. |
| -H, --dereference-command-line | Follow symbolic links listed on the command line. |
| --dereference-command-line-symlink-to-dir | Follow each command line symbolic link that points to a directory. |
| --hide=PATTERN | Do not list implied entries matching shell PATTERN (overridden by -a or -A). |
| --indicator-style=WORD | Append indicator with style WORD to entry names: none (default), slash (-p), file-type (--file-type), classify (-F). |
| -i, --inode | Print the index number of each file. |
| -I, --ignore=PATTERN | Do not list implied entries matching shell PATTERN. |
| -k, --kibibytes | Use 1024-byte blocks. |
| -l | Use a long listing format. |
| -L, --dereference | When showing file information for a symbolic link, show information for the file the link references rather than for the link itself. |
| -m | Fill width with a comma separated list of entries. |
| -n, --numeric-uid-gid | Like -l, but list numeric user and group IDs. |
| -N, --literal | Print raw entry names (don't treat e.g. control characters specially). |
| -o | Like -l, but do not list group information. |
| -p, --indicator-style=slash | Append "/" indicator to directories |
| -q, --hide-control-chars | Print ? instead of non graphic characters. |
| --show-control-chars | Show non graphic characters as-is (default unless program is 'ls' and output is a terminal). |
| -Q, --quote-name | Enclose entry names in double quotes. |
| --quoting-style=WORD | Use quoting style WORD for entry names: literal, locale, shell, shell-always, c, escape. |
| -r, --reverse | Reverse order while sorting. |
| -R, --recursive | List subdirectories recursively. |
| -s, --size | Print the allocated size of each file, in blocks. |
| -S | Sort by file size. |
| --sort=WORD | Sort by WORD instead of name: none (-U), extension (-X), size (-S), time (-t), version (-v). |
| --time=WORD | With -l, show time as WORD instead of modification time: "atime" (-u), "access" (-u), "use" (-u), "ctime" (-c), or "status" (-c); use specified time as sort key if --sort=time. |
| --time-style=STYLE | With -l, show times using style STYLE.  STYLE may be one of: "full-iso", "long-iso", "iso", "locale", "+FORMAT".  FORMAT is interpreted like 'date'; if FORMAT is "FORMAT1<newline>FORMAT2", FORMAT1 applies to non-recent files and FORMAT2 to recent files; if STYLE is prefixed with 'posix-', STYLE takes effect only outside the POSIX locale. |
| -t | Sort by modification time, newest first. |
| -T, --tabsize=COLS | Assume tab stops at each COLS instead of 8. |
| -u | With -lt:, sort by and show access time; with -l: show access time and sort by name; otherwise: sort by access time. |
| -U | Do not sort; list entries in directory order. |
| -v | Natural sort of (version) numbers within text. |
| -w, --width=COLS | Assume screen width COLS instead of current value. |
| -x | List entries by lines instead of by columns. |
| -X | Sort alphabetically by entry extension. |
| -Z, --context | Print any SELinux security context of each file. |
| -1 | List one file per line. |
| --help | Display a help message and exit. |
| --version | Display version information and exit. |

# SIZE Format

*SIZE* is an integer and optional unit (example: **10M** is **`10*1024*1024`**). Units are K, M, G, T, P, E, Z, Y (powers of 1024) or **KB**, **MB**, ... (powers of 1000).

Using color to distinguish file types is disabled both by default and with **--color=never**. With **--color=auto**, **ls** emits color codes only when standard output is connected to a terminal. The **LS_COLORS** environment variable can change the settings. Use the **dircolors** command to set it.

# Exit Status

|   |   |
| :-- | :-- |
| 0 | Everything is OK. |
| 1 | There were minor problems; for example, could not access a subdirectory. |
| 2 | There were serious problems; for example, a command-line option could not be accessed. |

# Examples

    ls -l

Lists the total files in the directory and subdirectories, the names of the files in the current directory, their permissions, the number of subdirectories in directories listed, the size of the file, and the date of last modification.

    ls -laxo

Lists files with permissions, shows hidden files, displays them in a column format, and suppresses group information.

    ls ~

List the contents of your home directory by adding a tilde after the ls command.

    ls /

List the contents of your root directory.

    ls ../

List the contents of the parent directory.

    ls */

List the contents of all subdirectories.

    ls -d */

Display a list of directories in the current directory.

    ls *.{htm,php,cgi}

List all files containing the file extension .htm, .php, or .cgi

    ls -ltr

List files sorted by the time they were last modified in reverse order (most recently modified files last).

    ls [aeiou]*

In the above example, only files that begin with a vowel (a, e, i, o, or u).

    ls myfile.txt 2>/dev/null

Silences or suppresses any error message if the ls command does not find the file.

Tip: Please also see our explanation of the ./ and ../ directories listed in the listing of files.

# Related commands

* [chmod](../chmod) — Change the permissions of files or directories.
* df — Report the amount of available disk space on file systems.
* diff — Identify the differences between two files.
* du — Report the amount of disk space used by a file or files.
* file — Determine a file's type.
* grep — Search for and output lines that match a specified pattern.
* stat — Display the status of a file or filesystem.
* tree — List the contents of a file hierarchy visually in a tree format.
