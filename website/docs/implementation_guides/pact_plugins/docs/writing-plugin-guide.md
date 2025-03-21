---
title: Guide to writing a Pact plugin
custom_edit_url: https://github.com/pact-foundation/pact-plugins/edit/main/docs/writing-plugin-guide.md
---
<!-- This file has been synced from the pact-foundation/pact-plugins repository. Please do not edit it directly. The URL of the source file can be found in the custom_edit_url value above -->

Pact plugins are essentially gRPC servers that run as child processes to the main Pact process (whether in a consumer
test or during provider verification). They are designed to be stateless and respond to requests from the Pact framework
that is running the tests. They can be written in any language that has gRPC support, but ideally should
be written in a language that has minimal system dependencies.

**IMPORTANT NOTE:** Please keep the end users in mind when selecting a language to write a plugin in. If you use, say
Java, that means any user who uses your plugin needs to have a JDK installed on their machines and CI servers, as well
as anything that verifies a Pact file created using that plugin even if the provider is written in a different language.

There are two prototype example plugins, one for [CSV](https://github.com/pact-foundation/pact-plugins/blob/main/plugins/csv) and one for [Protobuf](https://github.com/pact-foundation/pact-plugins/blob/main/plugins/protobuf). 
You can find examples of consumer and provider tests using these plugins in the [examples](https://github.com/pact-foundation/pact-plugins/blob/main/examples) in this repository.
The CSV plugin is written in Rust and the Protobuf one in Kotlin. 

## Plugin Interface

The first version of the plugin interface (version 1) supports extending Pact with [Content](https://github.com/pact-foundation/pact-plugins/blob/main/content-matcher-design.md) types, along with using Matchers and Generators to support flexible matching, and for adding new [Transport protocols](https://github.com/pact-foundation/pact-plugins/blob/main/protocol-plugin-design.md).

Content Matchers are for things like request/response bodies and message payloads and are based on specified MIME types, such as protobufs. 

Transports allow you to communicate these new content types over a different wire protocol, such as gRPC or Websockets.

Refer to for more details on the interface and gRPC methods that need to be implemented:

- [Content Matchers and Generators](https://github.com/pact-foundation/pact-plugins/blob/main/content-matcher-design.md)
- [Transport protocols](https://github.com/pact-foundation/pact-plugins/blob/main/protocol-plugin-design.md)

You can find the [proto file](https://github.com/pact-foundation/pact-plugins/blob/main/proto/plugin.proto) that defines the plugin interface in the proto directory. Your 
plugin will need to implement this interface.

When the plugin starts up, it needs to write a small JSON message to standard output that contains the port the plugin
is running on and an optional server key. The port should be one assigned by the operating system so there are no clashes
with other servers. The server key is reserved for use as a bearer token to restrict access to the
plugin from the Pact framework that started it. Ideally, the plugin gRPC server should bind to the loopback interface (127.0.0.1),
but this may not always be possible so if the plugin binds to all interfaces, the server key would provide a security
mechanism to not allow just any process to invoke the plugin methods.

You can see the prototype plugins doing this if you run their executable:

```commandline
$ ~/.pact/plugins/csv-0.0.0/pact-plugin-csv
{"port":35517, "serverKey":"56f7eb63-073b-429c-bff4-6ad336163067"}
```

Refer to the [Plugin drivers](https://github.com/pact-foundation/pact-plugins/blob/main/plugin-driver-design.md) for more details.

## Plugin manifest

Each plugin needs to have a manifest file named `pact-plugin.json` in JSON format that describes how the plugin should 
be loaded and any dependencies it requires. The format of the manifest is documented in [Plugin drivers](https://github.com/pact-foundation/pact-plugins/blob/main/plugin-driver-design.md). 
This file needs to be installed alongside your plugin executable files. Refer to the [CSV](https://github.com/pact-foundation/pact-plugins/blob/main/plugins/csv/pact-plugin.json) 
and [Protobuf](https://github.com/pact-foundation/pact-plugins/blob/main/plugins/protobuf/pact-plugin.json) manifest files for examples.

The important attribute in the manifest is the `entryPoint`. This is the executable that starts your plugin. The Protobuf
example also has an additional entry for Windows, because it uses batch files to start.

## Installing your plugin

By default, each plugin is installed (along with its manifest) in a directory named `<plugin name>-<version>` in 
the `.pact/plugins` directory in the users home directory. This default can be changed with the `PACT_PLUGIN_DIR`
environment file. `<plugin-name>` is the name of the plugin (corresponding to the name in the manifest) and `<version>`
if the version of the plugin. This way users can have different versions of your plugin installed.

Looking at the `.pact/plugins` on my machine we can see I have the two prototype plugins installed:

```commandline
$ ls -l ~/.pact/plugins/
total 8
drwxrwxr-x 2 ronald ronald 4096 Oct 18 14:09 csv-0.0.0
drwxrwxr-x 6 ronald ronald 4096 Oct 13 15:21 protobuf-0.0.0

$ ls -l ~/.pact/plugins/csv-0.0.0/
total 12376
-rwxrwxr-x 1 ronald ronald 12667032 Oct  6 12:04 pact-plugin-csv
-rw-rw-r-- 1 ronald ronald      237 Oct 18 14:09 pact-plugin.json

$ ls -l ~/.pact/plugins/protobuf-0.0.0/
total 28
drwxr-xr-x 2 ronald ronald  4096 Aug 27 12:40 bin
drwxr-xr-x 2 ronald ronald 12288 Oct 13 12:23 lib
-rw-rw-r-- 1 ronald ronald   352 Oct 11 11:07 pact-plugin.json
drwxrwxr-x 2 ronald ronald  4096 Oct 18 13:33 tmp
```

### Installing using the [pact-plugin-cli](https://github.com/pact-foundation/pact-plugins/tree/main/cli)

The `pact-plugin-cli` command can be used to manage plugins. To be able to install your plugin, the CLI tool requires:

* Plugin is released via GitHub releases with attached installation files.
* The plugin manifest file must be attached to the release and have the correct name and version.
* For single executable plugins, the executable attached to the release must be gzipped and named in the form `pact-${name}-plugin-${os}-${arch}(.exe?).gz`
  * `name` is the name from the plugin manifest file
  * `os` is the operating system (linux, windows, osx)
  * `arch` is the system architecture (x86_64, aarch64 for Apple M1. See https://doc.rust-lang.org/stable/std/env/consts/constant.ARCH.html)
  * Windows executables require `.exe` extension in the filename. Leave this out for Unix and OSX.
* For bundled plugins (like with Node.js or Java), you can use a Zip file. The file must be named `pact-${name}-plugin.zip` or `pact-${name}-plugin-${os}-${arch}.zip` if you have OS/arch specific bundles.

If you provide SHA256 files (with the same name but with `.sha256` appended), the installation command will check the downloaded
artifact against the digest checksum in that file. For example, the Protobuf plugin executable for Linux is named 
`pact-protobuf-plugin-linux-x86_64.gz` and the digest `pact-protobuf-plugin-linux-x86_64.gz.sha256`.

### If your plugin needs to use disk storage

By default, the plugins should be stateless. They will receive all the required data from Pact framework running the test.
However, if they need to use disk space, they should only write to files within the plugins installed directory. Some
versions of Unix or docker containers may not allow writing to the `/tmp` directory, and you won't know how the Pact tests
are going to be run.

If you look at the Protobuf directory above, you can see a `tmp` directory. This is where the proto file is written to be
passed to the protoc compiler and where the resulting proto descriptor is written. You should also clean up any files
written within the plugin directory.

When the plugin process is started, the current working directory will be set to the plugin's installed directory, so you
can use relative paths to load or write any files. The Protobuf plugin uses the relative path `./tmp` for the proto files.

## Plugin lifecycle

The plugin process will be started when the Pact framework detects that it is needed. This will be when a consumer test
runs that specifies that the plugin must be loaded or when a Pact file that needs the plugin is loaded to be verified. The
plugin driver library will control this. Ideally the plugin process will be kept running for as long as needed, but it may
also be started and stopped for each test. So don't rely on it being a long running process.
