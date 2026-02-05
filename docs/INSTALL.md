---
layout: default
title: "Installation and License Management"
---

# Installation and License Management

## Installation
Download the `.tar.gz` file of the latest QUBO++ release from the [**Latest Releases**](https://github.com/nakanocs/qbpp/releases/latest) page.
Extract the archive as follows:
```bash
$ tar xf qbpp_<arch>_<version>.tar.gz
```
The actual file name varies depending on the version you downloaded.
QUBO++ runs on Linux-based systems with the following CPUs:
* **amd64**: 64-bit Intel and AMD processors
* **arm64**: 64-bit ARM processors

## Setting Environment Variables
To compile and run QUBO++ programs, it is recommended to set the following environment variables:
* **`CPLUS_INCLUDE_PATH`**: Path to the directory containing the QUBO++ header files.
* **`LIBRARY_PATH`**: Path to the directory containing the QUBO++ shared libraries used during compilation and linking.
* **`LD_LIBRARY_PATH`:** Path to the directory containing the QUBO++ shared libraries required at runtime.
* **`QBPP_LICENSE_KEY`**: Your QUBO++ license key. If omitted, an anonymous trial license is used.
* **`PATH`**: Path to the directory containing the QUBO++ executable binaries.

To set these paths automatically each time a shell starts, add the following lines to the end of your **`~/.bashrc`**:
```bash
export QBPP_PATH=[QUBO++ install directory]

export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
export PATH=$QBPP_PATH/bin:$PATH

export QBPP_LICENSE_KEY=[Your QUBO++ license key]
```

## Lincense Management
The license is activated using the **`qbpp-license`** command as follows:
```bash
$ qbpp-license -a
```
This increments the activation count.
Each license has a limited number of allowed activations.
If `QBPP_LICENSE_KEY` is not set, the command attempts to activate an **Anonymous Trial License**.

To check the current status of your QUBO++ license, simply run:
```bash
$ qbpp-license
```
The current status can only be displayed for activated licenses.

If you want to move the license to another machine, you can deactivate it on this machine as follows:
```bash
$ qbpp-license -d
```
This decreases the activation count, and you can then activate the license on the other machine.

Since the license is node-locked, the activation information is stored on the machine, and re-activation is not necessary as long as the license has not expired.

> **WARNING**
> Each license key has a limited number of allowed activations and deactivations.
> Once the total number of activations and deactivations reaches this limit, you will no longer be able to activate or deactivate that license.

