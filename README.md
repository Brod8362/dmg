dmg
===
Conditional, multi-host dotflie manager

Features
========
- easily deploy your dotfiles with a single command
- re-use files with minor changes for different hosts
- quickly import existing files for a quick setup

Installation
============
> required: `python >= 3.10`
- Clone this repository, and place the `dmg` executable somewhere on your PATH.

Setup
=====
- Create a directory somewhere for your dotfile templates to be stored.
- Export the environment variables `DMG_ROOT=/path/to/files`, pointing at the directory you just created.
- `dmg -a import -f <file>` to import a file to be managed by dmg.
- Edit the files stored within your DMG_ROOT directory, and use `dmg -a deploy` to deploy them to their destinations. Any conditional directives will be parsed and applied.

Options
=======
> `-l {info, warning, error, debug, verify}` - set the log level

> `--dry` - dry run, do not perform destructive disk operations

> `-f FILE` - take in a file for input. used for many actions

> `-s DIR` - source directory. May use DMG_ROOT environment variable instead.

> `-a {deploy,import,parse,edit,delete,verify,diff}` - action of choice, see below

## Actions
`deploy`

Deploy all files from the source directory, or deploy a single file with `-f`.

- `dmg -a deploy`
- `dmg -a deploy -f ~/.config/my/config`

`import`

Import a new file to be managed by `dmg`. File must be specified with `-f`.

- `dmg -a import -f ~/.config/to/import`

`parse`

Parse a file already managed by `dmg`, and output it to stdout. File must be specified with `-f`.

- `dmg -a parse -f ~/.config/to/parse`

`edit`

Edit a file managed by `dmg`, and re-deploy if the syntax is OK. Will open using the EDITOR environment variable, or vim if it is not set. File must be specified with `-f`.

- `dmg -a edit -f ~/.config/to/edit`

`delete`

Remove a file so it will no longer be tracked by dmg. This does not delete the deployed version, just the dmg source version (stored in DMG_ROOT). File must be specified with `-f`.

- `dmg -a delete -f ~/.config/to/delete`

`verify`

Verify all files managed by dmg, and report their status.

- `dmg -a verify`

`diff`

View the differences between the parsed source version of a file and the deployed version. File must be specified with `-f`.

- `dmg -a diff -f ~/.config/to/diff`

Directives
==========
All dmg directives begin with `##$`, and are expected to be the *only* content on the line. The directives will NOT be included in the output file, so this should not cause any issues as far as various configuration formats go. (Though, many consider # to be a comment anyway)

> `hosts: arg1, arg2...`

name: hosts
description: the hosts that this file should be deployed to
argument(s): the hosts that this file should be deployed to

> `only: arg1, arg2...`

name: only
description: beginning of a block of text, that should only be included on the given hosts
argument(s): the hosts that this block of text should be included on

> `not: arg1, arg2...`

name: not
description: beginning of a block of text, that should be included on all hosts except given
argument(s): the hosts that this block of text should NOT be included on

> `end`

name: end
description: end block of text (to be used after only/not directive)
argument(s): none

Example
=======
Take the following (imaginary) configuration file:
```ini
blah_blah=something
foo=56
bar=42
exit=never
battery=yes
monitors=2
```

If you wanted to share mostly the same configuration between two systems (say - your laptop and your desktop), but have some minor differences, you could break the file up like this:

```ini
##$ hosts: my_desktop, my_laptop

blah_blah=something
foo=56
bar=42
exit=never

##$ only: my_laptop
battery=yes
##$ end
##$ only: my_desktop
monitors=2
##$ end
```
(please note that the extra lines are not necessary, but have been added for readability)

If this file is deployed to `my_desktop`, it will look like this:
```ini
blah_blah=something
foo=56
bar=42
exit=never

monitors=2
```
And when deployed to `my_laptop`, it will look like this:
```ini
blah_blah=something
foo=56
bar=42
exit=never

battery=yes
```

As you can see, the directives work a lot like the `#ifndef`, `#define`, and `#endif` preprocessor directives from C/C++.

You can also introduce more machines into the mix. For example, if you have a server, which does not need this config file, then you simply do not add it to the `hosts` directive. dmg will not deploy this file for that host.

You can also use the `not` directive, which is the inverse of the `only` directive. It will include the contents for all hosts *except* the ones listed in the directive arguments.