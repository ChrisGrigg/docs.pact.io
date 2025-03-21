---
title: CLI for managing Pact plugins
custom_edit_url: https://github.com/pact-foundation/pact-plugins/edit/main/cli/README.md
---
<!-- This file has been synced from the pact-foundation/pact-plugins repository. Please do not edit it directly. The URL of the source file can be found in the custom_edit_url value above -->

This project provides a command line interface to manage and install Pact plugins. It is a single executable binary.

## Command line interface

Running `pact-plugin-cli` without any options displays the standard help.

```console
❯ pact-plugin-cli 0.0.0
CLI utility for Pact plugins

USAGE:
    pact-plugin-cli [OPTIONS] <SUBCOMMAND>

OPTIONS:
    -d, --debug      Enable debug level logs
    -h, --help       Print help information
    -V, --version    Print version information
    -y, --yes        Automatically answer Yes for all prompts

SUBCOMMANDS:
    disable    Disable a plugin version
    enable     Enable a plugin version
    env        Print out the Pact plugin environment config
    help       Print this message or the help of the given subcommand(s)
    install    Install a plugin
    list       List installed plugins
    remove     Remove a plugin
```

### Displaying environment configuration

The `env` command will display any environment configuration that is being used.

```console
❯ pact-plugin-cli env
┌──────────────────┬─────────────────────┬────────────────────────────┐
│ Configuration    ┆ Source              ┆ Value                      │
╞══════════════════╪═════════════════════╪════════════════════════════╡
│ Plugin Directory ┆ $HOME/.pact/plugins ┆ /home/ronald/.pact/plugins │
└──────────────────┴─────────────────────┴────────────────────────────┘
```

### Listing installed plugins

Running the `list` command will list installed plugins.

```console
❯ pact-plugin-cli list
┌──────────┬─────────┬───────────────────┬───────────────────────────────────────────┬─────────┐
│ Name     ┆ Version ┆ Interface Version ┆ Directory                                 ┆ Status  │
╞══════════╪═════════╪═══════════════════╪═══════════════════════════════════════════╪═════════╡
│ csv      ┆ 0.0.1   ┆ 1                 ┆ /home/ronald/.pact/plugins/csv-0.0.1      ┆ enabled │
├╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ protobuf ┆ 0.1.7   ┆ 1                 ┆ /home/ronald/.pact/plugins/protobuf-0.1.7 ┆ enabled │
└──────────┴─────────┴───────────────────┴───────────────────────────────────────────┴─────────┘
```

### Enabling and disabling a plugin version

The `disable` command will disable a version of a plugin, while the `enable` command will enable it again.

```console
❯ pact-plugin-cli enable --help
pact-plugin-cli-enable 0.0.0
Enable a plugin version

USAGE:
    pact-plugin-cli enable <NAME> [VERSION]

ARGS:
    <NAME>       Plugin name
    <VERSION>    Plugin version. Not required if there is only one plugin version

OPTIONS:
    -h, --help    Print help information

```

It requires the plugin name and version. If there is only a single version, the version value can be omitted.

### Removing a plugin version

A particular version of a plugin can be removed with the `remove` command. It requires the plugin name and version. 
If there is only a single version, the version value can be omitted.

```console
❯ ./target/debug/pact-plugin-cli remove --help
pact-plugin-cli-remove 0.0.0
Remove a plugin

USAGE:
    pact-plugin-cli remove <NAME> [VERSION]

ARGS:
    <NAME>       Plugin name
    <VERSION>    Plugin version. Not required if there is only one plugin version

OPTIONS:
    -h, --help    Print help information
```

This will prompt to confirm the removal of the plugin, but that can be overridden with the `-y,-yes` option.

```console
❯ pact-plugin-cli -y remove csv
Removed plugin with name 'csv' and version '0.0.1'
```

### Installing a plugin

The `install` command can download and install a plugin from a GitHub release page. It will work out the correct
operating system and architecture required. If there are SHA256 digests of the download files, it will also check the digest
against the downloaded one.

You can point it to a release version, of use the latest link to download the latest version. I.e., for the Protobuf plugin,
https://github.com/pactflow/pact-protobuf-plugin/releases/tag/v-0.1.7 will install version 0.1.7, while
https://github.com/pactflow/pact-protobuf-plugin/releases/latest will install the latest version.

If the version of the plugin has already been installed, it will prompt to delete the existing one first. That can be
overridden with the `-y,-yes` option.

Example of installing the CSV plugin:

```console
❯ pact-plugin-cli -y install https://github.com/pact-foundation/pact-plugins/releases/tag/csv-plugin-0.0.1
Installing plugin csv version 0.0.1
Downloaded https://github.com/pact-foundation/pact-plugins/releases/download/csv-plugin-0.0.1/pact-csv-plugin-linux-x86_64.gz to /home/ronald/.pact/plugins/csv-0.0.1/pact-csv-plugin-linux-x86_64.gz
  [00:00:03] [#######################################################################################################################################################################] 3.43MiB/3.43MiB (973.64KiB/s, 0s)
Downloaded https://github.com/pact-foundation/pact-plugins/releases/download/csv-plugin-0.0.1/pact-csv-plugin-linux-x86_64.gz.sha256 to /home/ronald/.pact/plugins/csv-0.0.1/pact-csv-plugin-linux-x86_64.gz.sha256
  [00:00:00] [#############################################################################################################################################################################] 115B/115B (185.98KiB/s, 0s)
Installed plugin csv version 0.0.1 OK
```

## Installing

The CLI executable can be downloaded from the GitHub release page (i.e., https://github.com/pact-foundation/pact-plugins/releases/tag/pact-plugin-cli-v0.0.0).
There will be a file for each major OS and architecture. It just needs to be unzipped (using gunzip) and made executable on Unix.

```console
❯ wget https://github.com/pact-foundation/pact-plugins/releases/download/pact-plugin-cli-v0.0.0/pact-plugin-cli-linux-x86_64.gz
--2022-06-03 13:45:17--  https://github.com/pact-foundation/pact-plugins/releases/download/pact-plugin-cli-v0.0.0/pact-plugin-cli-linux-x86_64.gz
Resolving github.com (github.com)... 52.64.108.95
Connecting to github.com (github.com)|52.64.108.95|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/388319964/c0202f1f-b189-4e75-b45b-9d43acf1c632?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220603%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220603T034441Z&X-Amz-Expires=300&X-Amz-Signature=3a5c7f7a54288897a0f89c2c186e16da08cfcc6969c90f4f5535d4fc2dd0f68a&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=388319964&response-content-disposition=attachment%3B%20filename%3Dpact-plugin-cli-linux-x86_64.gz&response-content-type=application%2Foctet-stream [following]
--2022-06-03 13:45:17--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/388319964/c0202f1f-b189-4e75-b45b-9d43acf1c632?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220603%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220603T034441Z&X-Amz-Expires=300&X-Amz-Signature=3a5c7f7a54288897a0f89c2c186e16da08cfcc6969c90f4f5535d4fc2dd0f68a&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=388319964&response-content-disposition=attachment%3B%20filename%3Dpact-plugin-cli-linux-x86_64.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.110.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6186448 (5.9M) [application/octet-stream]
Saving to: ‘pact-plugin-cli-linux-x86_64.gz’

pact-plugin-cli-linux-x86_64.gz                            100%[=======================================================================================================================================>]   5.90M  3.89MB/s    in 1.5s    

2022-06-03 13:45:19 (3.89 MB/s) - ‘pact-plugin-cli-linux-x86_64.gz’ saved [6186448/6186448]

❯ gunzip pact-plugin-cli-linux-x86_64.gz
❯ chmod +x pact-plugin-cli-linux-x86_64  
❯ ./pact-plugin-cli-linux-x86_64 
pact-plugin-cli 0.0.0
CLI utility for Pact plugins

USAGE:
    pact-plugin-cli-linux-x86_64 [OPTIONS] <SUBCOMMAND>

OPTIONS:
    -d, --debug      Enable debug level logs
    -h, --help       Print help information
    -V, --version    Print version information
    -y, --yes        Automatically answer Yes for all prompts

SUBCOMMANDS:
    disable    Disable a plugin version
    enable     Enable a plugin version
    env        Print out the Pact plugin environment config
    help       Print this message or the help of the given subcommand(s)
    install    Install a plugin
    list       List installed plugins
    remove     Remove a plugin
```

### Installing using cargo

The executable can also be installed using the Rust Cargo command.

```console
❯ cargo install pact-plugin-cli
```

## Building

Executable is built using `cargo`.

### Requirements

Requires Rust 1.61 or later.
