## Requirements

#### Build tools:
- compiler: gcc or clang
- build automation tools: make + [cmake](https://cmake.org/)

#### Required libraries:
- [libyang](https://github.com/CESNET/libyang)
- [Google Protocol Buffers](https://github.com/google/protobuf)
- [protobuf-c](https://github.com/protobuf-c/protobuf-c)
- [libev](http://software.schmorp.de/pkg/libev.html)
- [libredblack](http://libredblack.sourceforge.net/) or [GNU libavl](http://adtinfo.org/) (either of these two)

#### (Optional) Tools for running tests and building documentation:
- [CMocka](https://cmocka.org/)
- [valgrind](http://valgrind.org/)
- [doxygen](http://www.doxygen.org)

#### (Optional) Bindings for other languages:
[Swig](http://www.swig.org/) must be installed. Bindigs are generated during `make` phase.
- Python bindings require `python-dev` to be installed.
- Lua bindings require `lua5.2` to be installed.

#### (Optional) Netopeer2 NETCONF server
Can be installed to enable remote management via NETCONF. Requires [libnetconf2](https://github.com/CESNET/libnetconf2) library. Follow the instructions on [Netopeer2](https://github.com/CESNET/Netopeer2) site, or have a look at the [Dockerfile](deploy/docker/sysrepo-netopeer2/Dockerfile) for Sysrepo & Netopeer2 integration.

#### Installation of required libraries:
On Debian-like Linux distributions:
- `apt-get install git cmake build-essential bison flex libpcre3-dev libev-dev libavl-dev libprotobuf-c-dev protobuf-c-compiler`
- (optional) `apt-get install valgrind swig python-dev lua5.2`
- CMocka and libyang need to be installed from sources

On FreBSD:
- `pkg install cmake git protobuf protobuf-c libev libredblack`
- CMocka and libyang need to be installed from sources

On Mac OS X:
- `brew cmake protobuf protobuf-c libev`
- CMocka, libyang and libredblack need to be installed from sources


## Installation of required libraries from sources

#### CMocka:
```
$ git clone git://git.cryptomilk.org/projects/cmocka.git
$ cd cmocka
$ mkdir build; cd build
$ cmake -DCMAKE_BUILD_TYPE=Debug ..
$ make
# make install
```

#### Libyang:
```
# apt-get install libpcre3-dev
$ git clone https://github.com/CESNET/libyang.git
$ cd libyang; mkdir build; cd build
$ cmake ..
$ make
# make install
```

#### Google Protocol Buffers (not needed if already installed from packages):
```
# apt-get install autoconf libtool
$ git clone https://github.com/google/protobuf.git
$ cd protobuf
$ ./autogen.sh
$ ./configure
$ make
# make install
```

#### Protobuf-c (not needed if already installed from packages):
```
$ git clone https://github.com/protobuf-c/protobuf-c.git
$ cd protobuf-c
$ ./autogen.sh && ./configure --prefix=/usr 
$ make 
# make install
```

#### libredblack (not needed if libavl/libredbalck already installed from packages):
```
$ git clone https://github.com/sysrepo/libredblack.git
$ cd libredblack
$ ./configure
$ make
# make install
```

## Building sysrepo
1) Get the source code and prepare the build directory:
```
$ git clone https://github.com/sysrepo/sysrepo.git
$ cd sysrepo
$ mkdir build; cd build
```
2 a) Configure build for production use (Release build):
```
$ cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
```
2 b) Configure build for testing and development (Debug build):
```
$ cmake -DCMAKE_BUILD_TYPE=Debug ..
```
3) Build:
```
$ make
```
4 a) Run unit tests (some of them applicable only to Debug build)
```
$ ctest
```
4 b) Install
```
$ make install
```
5) (optional) Build Doxygen documentation:
```
make doc
```


## Useful CMake options
#### Changing build mode:
Sysrepo supports two build modes:
- Release - generates the library and executables for the production use, without any debug information and with compiler optimization of the code enabled. Default for the master branch.
- Debug - generates the library and executables with the debug information and disables any compiler optimizations of the code, enables all unit tests. Default for the development branches.

To change the build mode use `CMAKE_BUILD_TYPE` variable as follows: `cmake -DCMAKE_BUILD_TYPE=Release ..`

#### Changing install path:
To change the location where the library, headers and any other files are installed (default is `/usr/local`), use `CMAKE_INSTALL_PREFIX` variable as follows: `cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..`

#### Changing repository location:
Sysrepo stores all YANG models and corresponding data files in so-named *repository location*. By default, the repository location is set to `/etc/sysrepo` for Release build and `tests/data` (relatively to the directory from which the cmake command is executed) for Debug build. To change default values, use `REPOSITORY_LOC` variable as follows: `cmake -DREPOSITORY_LOC:PATH=/etc/sysrepo ..`

#### Changing plugins directory location:
All sysrepo plugins should be placed into *plugins directory*. This defaults
to `${CMAKE_INSTALL_LIBDIR}/sysrepo/plugins/` (e.g. `/usr/local/lib/sysrepo/plugins/`) and can be changed by `PLUGINS_DIR` variable as follows: `cmake -DPLUGINS_DIR:PATH=/opt/sysrepo/plugins ..`

#### Building with / without examples:
By default, some [example programs](examples/) are built with sysrepo and several [example YANG modules](examples/yang/) are installed into sysrepo repository, along with some meaningless data. If you wish to not build and install them, use `BUILD_EXAMPLES` varibale as follows: `cmake -DBUILD_EXAMPLES:BOOL=FALSE ..`

## Using sysrepo
By installation, three main parts of sysrepo are installed on the system: **sysrepoctl tool**, **sysrepo library** and **sysrepo daemon**.

#### Using sysrepoctl & sysrepocfg tools
`sysrepoctl` is a tool for the management of YANG modules installed in sysrepo. It can be used for installing of new YANG modules to sysrepo, uninstalling existing ones, listing current state of installed modules, enabling / disabling of YANG features within the module or changing access permissions.

`sysrepocfg` can be used for importing data from / to sysrepo, as well as for editing startup or running configuration of specified module in preferred text editor.

Detailed usage of the tools can be displayed by executing `sysrepoctl -h` or `sysrepocfg -h`. Here are some examples of the usage:

Install a new module by specifying YANG file, ownership and access permissions:

`sysrepoctl --install --yang=/home/user/ietf-interfaces.yang --owner=admin:admin --permissions=644`

Change the ownership and permissions of an existing YANG module:

`sysrepoctl --change --module=ietf-interfaces --owner=admin:admin --permissions=644`

Enable a feature within a YANG module:

`sysrepoctl --feature-enable=if-mib --module=ietf-interfaces`

Export (dump) startup datastore data of a YANG module into a file in XML format:

`sysrepocfg --export=dump_file.xml --format=xml --datastore=startup ietf-interfaces`

Import startup datastore data of a YANG module from a file in XML format:

`sysrepocfg --import=dump_file.xml --format=xml ietf-interfaces`


#### Using sysrepo library in your application
Sysrepo library is an interface between sysrepo datastore and northbound and southbound applications. To use it, you need to link `libsysrepo` to your application and include sysrepo public header file in the source that needs to use it:
```
#include <sysrepo.h>
...
sr_conn_ctx_t *conn = NULL;
sr_connect("application_name", SR_CONN_DEFAULT, &conn);
...
```
See [examples directory](examples/) for more usage examples.

#### Starting sysrepo daemon
Sysrepo deamon provides the functionality of the datastore on the system and should normally be automatically started by system startup. However, auto-start is not configured by cmake install operation and you need to configure it yourself, accroding to the guidelines of your system.

Sysrepo deamon can be started by executing of the following command:
```
sysrepod
```

The daemon accepts several arguments aimed for debugging. You can display them by executing `sysrepod -h`:
```
$ sysrepod -h
sysrepod - sysrepo daemon, version 0.1.12

Usage:
  sysrepod [-h] [-d] [-v <level>]

Options:
  -h            Prints this usage help.
  -d            Debug mode - daemon will run in the foreground and print logs to stderr instead of syslog.
  -v <level>    Sets verbosity level of logging:
                    0 = all logging turned off
                    1 = log only error messages
                    2 = (default) log error and warning messages
                    3 = log error, warning and informational messages
                    4 = log everything, including development debug messages
```

#### Starting sysrepo plugin daemon
Sysrepo plugin daemon loads all plugins (shared libraries) located in the *plugins directory*. It works similarly to the main sysrepo damon described above (and also accepts the same arguments) and can be started by executing of the following command:
```
sysrepo-plugind
```

#### Subscribing to running datastore
As described in the documentation section named Datastores & Sessions, to enable a configuration subtree in the running datastore, some application needs to subscribe for the changes in the subtree in sysrepo datastore. If no application has subscribed for specified subtree, it would have no data in the running datastore and all attempts to modify the subtree in the running datastore will result into an error.

For testing purposes, you can use [application_example](examples/application_example.c) or [application_changes_example](examples/application_changes_example.c) from the [examples](examples/) subdirectory. These applications subscribe for chnges in the module specified as the argument and print the new configuration after it has changed:
```
examples/application_example ietf-interfaces
```
