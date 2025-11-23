# QUBO++ Installation/License Activation

## Installation
Simply extract the `.tar.gz` file as follows:
```bash
$ tar xf qbpp_amd64_2025.11.23.tar.gz
```
The file name may differ depending on the version you downloaded.
QUBO++ runs on Linux-based systems with the following CPUs:
* amd64: 64-bit Intel and AMD processors
* arm64: 64-bit ARM processors

## Setting Environment Variables
It is recommended to set the following environment variables:
* `CPLUS_INCLUDE_PATH`: Specifies the include path that contains the QUBO++ header files.
* `LIBRARY_PATH`: Specifies the library path that contains the QUBO++ shared libraries for compiling and linking QUBO++ programs.
* `LD_LIBRARY_PATH`: Specifies the library path that contains the QUBO++ shared libraries for executing QUBO++ programs.
* `QBPP_LICENSE_KEY`: Specifies your license key. If omitted, an anonymous trial license is assumed.
* `PATH`: Specifies the directory containing QUBO++ executable binaries.

To set these paths automatically when a shell starts, add the following lines to the end of your `~/.bashrc`:
```bash
export QBPP_PATH=[QUBO++ install directory]

export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
export PATH=$QBPP_PATH/bin:$PATH

export QBPP_LICENSE_KEY=[Your QUBO++ license key]
```

## Lincense Activation
The license is activated using the `qbpp-license` command as follows:
```bash
$ qbpp-license -a
```
This increments the activation count.
Each license has a limited number of allowed activations.
If `QBPP_LICENSE_KEY` is not set, the command attempts to activate an anonymous trial license.
Since the license is node-locked, the activation information is stored on the machine, and it is not necessary to run this command again
as long as the license has not expired.

If you want to move the license to another machine, you can deactivate it on this machine as follows:
```bash
$ qbpp-license -d
```
This decreases the activation count, and you can then activate the license on the other machine.
