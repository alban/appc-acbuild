# acbuild

acbuild is a command line utility to build and modify App Container Images 
(ACIs), the container image format defined in the 
[App Container (appc) spec](https://github.com/appc/spec).

## Rationale

Dockerfiles are powerful and feature useful concepts such as build layers, 
controlled build environment. At the same time, they lack flexibility 
(impossible to extend, re-use environment variables) and don't play nicely 
with the appc spec and Linux toolchain (e.g. shell, makefiles)

`acbuild` is a command-line tool that natively supports ACI builds and integrates 
well with shell, makefiles and
other Unix tools.

## Installation

### Runtime dependencies

acbuild requires a handful of commands be available on the system on 
which it's run:

- `systemd-nspawn`
- `cp`
- `mount`
- `modprobe`
- `tar`

### Build from source

Currently the only way to install `acbuild` is to build from source. 
Follow these steps to do so:

1. Grab the source code for `acbuild` by `git clone`ing the source repository:
   ```
   cd ~
   git clone git@github.com:appc/acbuild
   ```

2. Run the `build` script from the root source repository directory:
   ```
   cd acbuild
   ./build
   ```

3. A `bin/` directory will be created that contains the `acbuild` tool. To make
   sure your shell can find this executable, append this directory to your
   environment's `$PATH` variable. You can do this in your `.bashrc` or similar
   file, for example:
   ```
   vi ~/.bashrc
   ```

and put the following lines at the end of the file:
   ```
   export ACBUILD_BIN_DIR=~/acbuild/bin
   export PATH=$PATH:$ACBUILD_BIN_DIR
   ```

## Usage

A build with `acbuild` is explicitly started with `begin` and finished with
`end`. While a build is in progress the current ACI is stored expanded in the
current working directory at `.acbuild.tmp`. A build can be started with an
empty ACI, or an initial ACI can be provided.

The following commands are supported:

* `acbuild begin`

  Begin a build.

* `acbuild end ACI_PATH`

  End a build, writing the resulting ACI to `ACI_PATH`.

* `acbuild abort`

  Abort the current build, throwing away any changes since `begin` was called.

* `acbuild annotation add NAME VALUE`

  Updates the ACI to contain an annotation with the given name and value. If the
  annotation already exists, its value will be changed.

* `acbuild annotation remove NAME`

  Removes the annotation with the given name from the ACI.

* `acbuild dependency add IMAGE_NAME --image-id sha512-... --label env=canary`

  Updates the ACI to contain a dependency with the given name. If the dependency
  already exists, its values will be changed.

* `acbuild dependency remove IMAGE_NAME`

  Removes the dependency with the given image name from the ACI.

* `acbuild environment add NAME VALUE`

  Updates the ACI to contain an environment variable with the given name and
  value. If the variable already exists, its value will be changed.

* `acbuild environment remove NAME`

  Removes the environment variable with the given name from the ACI.

* `acbuild label add NAME VALUE`

  Updates the ACI to contain a label with the given name and value. If the label
  already exists, its value will be changed.

* `acbuild label remove NAME`

  Removes the label with the given name from the ACI.

* `acbuild mount add NAME PATH`

  Updates the ACI to contain a mount point with the given name and path. If the
  mount point already exists, its path will be changed.

* `acbuild mount remove NAME`

  Removes the mount point with the given name from the ACI.

* `acbuild port add NAME PROTOCOL PORT`

  Updates the ACI to contain a port with the given name, protocol, and port. If
  the port already exists, its values will be changed.

* `acbuild port remove NAME`

  Removes the port with the given name from the ACI.

* `acbuild copy PATH_ON_HOST PATH_IN_ACI`

  Copy a file or directory from the local filesystem into the ACI.

* `acbuild run CMD [ARGS]`

  Run a given command in the ACI.

* `acbuild set-name ACI_NAME`

  Changes the name of the ACI in its manifest.

* `acbuild set-group GROUP`

  Set the group the app will run as inside the container.

* `acbuild set-user USER`
  
  Set the user the app will run as inside the container

* `acbuild set-exec CMD [ARGS]`

  Sets the exec command in the ACI's manifest.

### acbuild run

`acbuild run` builds the root filesystem with any dependencies the ACI has
using overlayfs, and then executes the given command using systemd-nspawn. The
current ACI being built is the upper level in the overlayfs, and thus modified
files that came from the ACI's dependencies will be copied into the ACI. More
information on this behavior is available
[here](https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt).

`acbuild run` requires overlayfs if the ACI being operated upon has
dependencies.

`acbuild run` also requires root.

## Planned features

### Context-free mode

There are scenarios in which it is not convenient to need to call `begin` and
`end`, the most obvious being when a single change is made to an existing ACI.
A flag will be added to allow every subcommand to be performed on a given ACI,
instead of looking for a current build in progress.

### Image signing

It would be convenient if the appropriate gpg keys could be passed into `acbuild
end`, and a ASC file would then be produced in addition to the ACI file.

### Squash

`acbuild squash`: fetch all the dependencies for the given image and squash them
together into the ACI without dependencies.

### Image fetching with begin

Pass in an image name, instead of a path to the ACI for `acbuild begin`, and the
image will be fetched and used as the starting point for the build.

### Alternate execution engine

Support multiple execution engines, notably runc.

## Examples

Use apt-get to install nginx.

```
acbuild begin
acbuild add-dep quay.io/fermayo/ubuntu
acbuild run -- apt-get update
acbuild run -- apt-get -y install nginx
acbuild end ubuntu-nginx.aci
```

## Related work

- https://github.com/sgotti/baci
- https://github.com/appc/spec/tree/master/actool - particularly the `build` and
  `patch-manifest` subcommands. `acbuild` may subsume such functionality,
  leaving `actool` as a validator only.


