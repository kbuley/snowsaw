<p align="center"><img src="https://cdn.rawgit.com/arcticicestudio/snowsaw/develop/assets/snowsaw-banner.svg"/></p>

<p align="center"><img src="https://assets-cdn.github.com/favicon.ico" width=24 height=24/> <a href="https://github.com/arcticicestudio/snowsaw/releases/latest"><img src="https://img.shields.io/github/release/arcticicestudio/snowsaw.svg"/></a> <a href="https://github.com/arcticicestudio/snowsaw/releases/latest"><img src="https://img.shields.io/badge/pre--release---_-blue.svg"/></a> <img src="https://www.python.org/static/favicon.ico" width=24 height=24/> <img src="https://img.shields.io/badge/Python-3.5+-blue.svg"/></p>

<p align="center">A lightweight, plugin-driven and simple configurable dotfile bootstrapper.</p>

---

It does less than you think, because version control systems do more than you think.  
Designed to be self-contained and extensible with no external dependencies and no installation required.

  - [Getting started](#getting-started)
    - [Integration](#integration)
      - [Add the submodule](#add-the-submodule)
      - [Create a bootstrap script](#create-a-bootstrap-script)
    - [Version Update](#version-update)
  - [Design Concept](#design-concept)
    - [snowblocks](#snowblocks)
    - [Repository Structure](#repository-structure)
  - [Plugins](#plugins)
    - [Plugins API](#plugin-api)
  - [CLI](#cli)
  - [Configuration](#configuration)
    - [Core Tasks](#core-tasks)
      - [`link`](#link)
      - [`shell`](#shell)
      - [`clean`](#clean)
    - [Defaults](#defaults)
  - [Development](#development)
    - [Debugging](#debugging)
      - [JetBrains PyCharm](#jetbrains-pycharm)
    - [Contribution](#contribution)
  - [Credits](#credits)

## Getting started
Plase make sure to read the [design concept](#design-concept) and [configuration](#configuration) documentation sections before integrating snowsaw to understand the way it works.

### Integration
#### Add the submodule
Create the base directory for snowblocks and add snowsaw with the latest stable version as a submodule to your dotfile repository:
```
mkdir snowblocks
git submodule add https://github.com/arcticicestudio/snowsaw .snowsaw
```
This command will add the snowsaw project at the main development branch `develop`, but it is recommened to use a stable release version by running
```sh
cd .snowsaw
git checkout v0.2.0
cd ..
```
and commit the changes in your dotfile repository to lock it on the specified version tag.  
The "[Update the version](#update-the-version)" section contains more information on how to update to another version at any time.

The `develop` branch will always include the latest commits, but use it on your own risk as it may raise unexpected error or compatibility issues when fetching the latest changes without ensuring that it runs with your current dotfile configurations.

The latest changes from the `develop` branch can be simply fetched by running
```sh
git submodule update --remote .snowsaw
```

#### Create a bootstrap script
It is recommened to create a `bootstrap` script that calls the [`snowsaw`](https://github.com/arcticicestudio/snowsaw/blob/master/bin/snowsaw) binary with the required parameters.  
Information about available options and environment variables can be found in the [CLI](#cli) documentation.
```sh
#!/usr/bin/env bash
set -e

SNOWSAW_DIR=".snowsaw"
SNOWSAW_BIN="bin/snowsaw"

SNOWBLOCKS_BASE_DIR_NAME="snowblocks"
SNOWBLOCKSDIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)/$SNOWBLOCKS_BASE_DIR_NAME"

"${SNOWSAW_DIR}/${SNOWSAW_BIN}" -s "${SNOWBLOCKSDIR}" "${@}"
```
The `${@}` allows to additionally specify supported options on the terminal for a dynamic script execution.

### Version Update
The snowsaw version can be simply updated by checking out to the desired version tag inside the submodule repository:
```sh
cd .snowsaw
git checkout <TAG>
cd ..
```

## Design Concept
### <img src="https://cdn.rawgit.com/arcticicestudio/snowsaw/develop/assets/icon-snowblocks.svg"/> snowblocks
A `snowblock` is a named directory that represents a topic area.  
Every valid `snowblock` contains a `snowblock.json` configuration file.  
All `snowblock` directories are placed in one base directory, defaults to `<DOTFILE_REPOSITORY_ROOT>/snowblocks`, which will be processed recursively.

This design allows a modular structured dotfile repository where each topic can be represented as a `snowblock` instead of placing all files and folders without any logical division in the dotfile repository root.  
The structure plays well with the snowsaw feature that allows to specify one configuration file to only process a single `snowblock`.

### Repository Structure
snowsaw does not need any specific repository structure expect the base snowblocks directory. A dotfile repository may also contain more than one base snowblock directory. This way the repository can be structured even more fine-grained.

The dotfile repository [Igloo](https://github.com/arcticicestudio/igloo) may be used as a repository structure reference.

```
<DOTFILE_REPOSITORY_ROOT>
|-- .git
|-- .github
|-- .snowsaw
|-- assets
|-- snowblocks
    |-- atom
        |-- config.cson
        |-- projects.cson
        |-- snowblock.json
    |-- git
        |-- gitconfig
        |-- git-commit-message
        |-- gitingore
        |-- snowblock.json
    |-- vim
        |-- snowblock.json
        |-- vimrc
|-- .gitignore
|-- .gitmodules
|-- CHANGELOG.md
|-- LICENSE.md
|-- README.md
|-- bootstrap
```
> Example directory tree of a dotfile repository

## Plugins
snowsaw is designed as a plugin system to be easily extensible.
The [plugin API](#plugin-api) also allows users to implement their own plugins for custom tasks.
Tasks are detected as a `directive` where each plugin can handle one or more directive.

The snowsaw core plugins provide tasks to
  - [`link`](#link) files and folders
  - execute [`shell`](#shell) commands
  - [`clean`](#clean) directories of broken symbolic links

All core plugins are loaded by default, but can be disabled by using the `--disable-core-plugins` terminal option.

A list of available plugins can be found in the [project wiki](https://github.com/arcticicestudio/snowsaw/wiki/Plugins).

All core plugins have been implemented following the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle) and [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy).  

### Plugin API
Plugins are implemented as subclasses of [`snowsaw.Plugin`](https://github.com/arcticicestudio/snowsaw/blob/master/snowsaw/plugin.py).  
They must implement the methods
```python
can_handle()
handle()
```

The `can_handle()` method should return `True` if the plugin can handle an action with the given name.  
The `handle()` method should process the task and return whether or not it completed successfully.

Plugins are loaded using the `--plugin` and `--plugin-dir` terminal option, using either absolute paths or paths relative to the base directory.

The [core plugins](https://github.com/arcticicestudio/snowsaw/tree/master/snowsaw/plugins) can be used as a reference to implement custom plugins.

## CLI
snowsaw supports to specify CLI terminal parameters to dynamically control the execution.  
A terminal help page can be shown by using the `-h`/`--help` option.

| Option | Parameter(s) | Required | Description |
| --- | --- | --- | --- |
| `-Q`, `--super-quiet` | - | No | Suppress almost all output. |
| `-q`, `--quiet` | - | No | Suppress most output. |
| `-v`, `--verbose` | - | No | Enable verbose output. |
| `-s`, `--snowblocks-directory` | `SNOWBLOCKSDIR` | Yes | Base snowblock directory to run all tasks of. |
| `-c`, `--config-file` | `CONFIGFILE` | No | Run tasks for the specified snowblock. |
| `-p`, `--plugin` | `PLUGIN` | No | Load `PLUGIN` as a plugin. |
| `--disable-core-plugins` | - | No | Disable all core plugins. |
| `--plugin-dir` | `PLUGIN_DIR` | No | Load all plugins in `PLUGIN_DIR`. |

## Configuration
snowsaw uses [JSON](http://json.org) configuration files to specify tasks on how to set up your dotfiles.

A configuration file is a array of tasks, where each task is a dictionary that contains a command name mapping to data for that command. Tasks are run in the order in which they are specified. Commands within a task do not have a defined ordering.

### Core Tasks
#### `link`
Links specify how files and directories should be symbolically linked. If desired, items can be specified to be forcibly linked, overwriting existing files if necessary. Environment variables in paths are automatically expanded.

##### Format
Links are specified as a dictionary mapping targets to source locations. Source locations are specified relative to the base `snowblock` directory that is specified as terminal parameter. Directory names should not contain a trailing `/` character.

Links support an optional extended configuration. In this type of configuration, instead of specifying source locations directly, targets are mapped to extended configuration dictionaries.  
These dictionaries support the following options:

| Option | Values | Default Value | Required | Description |
| --- | --- | --- | --- | --- |
| `create` | `true`, `false` | `false` | No | Specifies if the parent directory should be created if necessary. |
| `force` | `true`, `false` | `false` | No | Specifies if the file or directory should be forcibly linked. **This can cause irreversible data loss! Use with caution!** |
| `hosts` | `dict` | `{}` | No |  Contains key-value entries with hostnames and their associated target path this link should be processed for. Links with an empty dictionary will be processed irrespective of the host.<br><br>The `default` hostname can be specified as fallback target if non of the given hostnames matched. **Note that this will not work for target hosts named `default`!** |
| `path` | `string`, `null` | `null` | No |  The path to map the source path. If the path is omitted or `null`, snowsaw will use the basename of the destination, with a leading `.` stripped if present. |
| `relink` | `true`, `false` | `false` |  No | Specifies if incorrect symbolic links should be automatically overwritten. |
| `relative` | `true`, `false` | `false` |  No | Specifies if the symbolic link should have a relative path. |

##### Example
```json
[
  {
    "link": {
      "~/.gitconfig": {
        "create": true,
        "hosts": {
          "archlinux-home": "gitconfig.home",
          "archlinux-work": "gitconfig.work",
          "default": "gitconfig.base"
        }
      },
      "~/.gitconfig_auth": {
        "path": "gitconfig_auth.local"
      },
      "~/.gitignore": {
        "force": true,
        "relink": true,
      },
      "~/.git-commit-message": {
        "relative": true
      }
    }
  }
]
```
If the source location is omitted or set to `null`, snowsaw will use the basename of the destination, with a leading `.` stripped if present.

#### `shell`
The shell task specifies shell commands to be run. Shell tasks are run in the base `snowblock` directory that is specified as terminal parameter.

##### Format
Shell tasks can be specified in several different ways. The simplest way is just to specify a command as a string containing the command to be run.

Another way is to specify a two element array where the first element is the shell command and the second is an optional human-readable description.

Shell tasks support an extended syntax as well, which provides more fine-grained control. A command can be specified as a dictionary that contains the following options:

| Option | Values | Default Value | Required | Description |
| --- | --- | --- | --- | --- |
| `host` | `string` | - | No | The hostname of the machine this script should be executed on. Not supplying a host will result in the shell command executing on every host.<br><br>The hostname '-' can be specified as a default target that will match all hosts. |
| `command` | `string` | - |  Yes | The command to be run. |
| `description` | `string` | - | No |  A human-readable description. |
| `stdin` | `true`, `false` | `false` | No |  Specifies if the standard input stream is enabled. |
| `stdout` | `true`, `false` | `false` | No |  Specifies if the standard output stream is enabled. |
| `stderr` | `true`, `false` | `false` | No |  Specifies if the standard error stream is enabled. |

##### Example
```json
[
  {
    "shell": [
      "mkdir -p ~/yogurt",
      ["mkdir -p ~/yogurt", "Creating yogurt folder"],
      {
        "command": "mkdir -p ~/coconut",
        "description": "Creating coconut folder",
        "stderr": true,
        "stdin": true,
        "stdout": true
      }
    ]
  }
]
```

#### `clean`
Clean tasks specify directories that should be checked for broken symbolic links. These broken links are removed automatically. Only broken links that point to the dotfiles directory are removed.

##### Format
Clean commands are specified as an array of directories to be cleaned.

##### Example
```json
[
  {
    "clean": ["~"]
  }
]
```

### Defaults
Default options for plugins can be specified so that options don't have to be repeated many times.

Defaults apply to all tasks that follow setting the defaults. Defaults can be set multiple times where each change replaces the defaults with a new set of options.

##### Format
Defaults are specified as a dictionary mapping action names to settings, which are dictionaries from option names to values.

##### Example
```json
[
  {
    "defaults": {
      "create": true,
      "relink": true
    }
  }
]
```

## Development
[![](https://img.shields.io/badge/Changelog-0.2.0-blue.svg)](https://github.com/arcticicestudio/snowsaw/blob/v0.2.0/CHANGELOG.md) [![](https://img.shields.io/badge/Workflow-gitflow--branching--model-blue.svg)](http://nvie.com/posts/a-successful-git-branching-model) [![](https://img.shields.io/badge/Versioning-ArcVer_0.8.0-blue.svg)](https://github.com/arcticicestudio/arcver)

### Debugging
#### JetBrains PyCharm
snowsaw is developed using one of the awesome tools from JetBrains called [PyCharm](https://www.jetbrains.com/pycharm).
The project files are located in the [`.idea`](https://github.com/arcticicestudio/snowsaw/tree/develop/.idea) directory.

The included [run/debug configurations](https://www.jetbrains.com/help/pycharm/run-debug-configurations.html) for the script must be manually adjusted to match the paths to the main [`snmowsaw`](https://github.com/arcticicestudio/snowsaw/blob/develop/bin/snowsaw) script and the path parameter for the `-s` / `--snowblocks-directory` CLI option.
![][scrot-readme-debugging-run-configuration]

snowsaw can [run](https://www.jetbrains.com/help/pycharm/running-applications.html) in the debug mode by using the bug icon or from the menu via *Run* -> *Debug 'snowsaw'*.
![][scrot-readme-debugging-run]
![][scrot-readme-debugging-run-menu]

The [debug window](https://www.jetbrains.com/help/pycharm/debug-tool-window.html) will automatically toggle to show [*Variables*](https://www.jetbrains.com/help/pycharm/debug-tool-window-variables.html) for defined debug breakpoints marked in the editor gutter.
![][scrot-readme-debugging-code]
![][scrot-readme-debugging-variables]

### Contribution
Please report issues/bugs, feature requests and suggestions for improvements to the [issue tracker](https://github.com/arcticicestudio/snowsaw/issues).

## Credits
snowsaw is based on the awesome [Dotbot](https://github.com/anishathalye/dotbot) project by [@anishathalye](https://github.com/anishathalye) as a customized fork for my personal dotfiles repository [Igloo](https://github.com/arcticicestudio/igloo).

<p align="center"><img src="https://cdn.rawgit.com/arcticicestudio/nord/develop/src/assets/banner-footer-mountains.svg" /></p>

<p align="center"><img src="https://img.shields.io/badge/License-MIT-blue.svg"/></p>

[scrot-readme-debugging-run-configuration]: https://raw.githubusercontent.com/arcticicestudio/snowsaw/develop/assets/scrot-readme-debugging-run-configuration.png
[scrot-readme-debugging-run]: https://raw.githubusercontent.com/arcticicestudio/snowsaw/develop/assets/scrot-readme-debugging-run.png
[scrot-readme-debugging-run-menu]: https://raw.githubusercontent.com/arcticicestudio/snowsaw/develop/assets/scrot-readme-debugging-run-menu.png
[scrot-readme-debugging-variables]: https://raw.githubusercontent.com/arcticicestudio/snowsaw/develop/assets/scrot-readme-debugging-variables.png
[scrot-readme-debugging-code]: https://raw.githubusercontent.com/arcticicestudio/snowsaw/develop/assets/scrot-readme-debugging-code.png
