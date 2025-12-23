---
layout: default
title: "WSL"
---

# Installing QUBO++ on WSL
On Windows 11, QUBO++ can be used through WSL (Windows Subsystem for Linux), which allows Linux programs to run natively on Windows.

This document explains how to install WSL, required libraries, and QUBO++, and how to compile and run sample programs.

## Installing WSL
On Windows 11, WSL can be installed from Windows PowerShell.
Open PowerShell as Administrator and execute the following command:
```
C:\WINDOWS\System32> wsl --install
Provisioning the new WSL instance Ubuntu
This might take a while...
Create a default Unix user account: [user account name]
New password: [your password]
Retype new password: [your password]
passwd: password updated successfully
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```
Enter your [user account name] and [your password] when prompted.
This installs WSL 2 and an Ubuntu-based Linux system running on Windows.

After the installation is complete, update and upgrade the system software inside WSL:
```bash
$ sudo apt update
$ sudo apt upgrade -y
```

## Installing C++ compiler, Boost, and oneTBB
QUBO++ requires a C++ compiler, the Boost library, and oneTBB.

Install them using the following command:
```bash
$ sudo apt install -y build-essential libboost-all-dev libtbb-dev
```

## Installing QUBO++
Download the `.tar.gz` file of the latest QUBO++ release from the [Latest Release](https://github.com/nakanocs/qbpp/releases/latest) page.

If the file is downloaded to your Windows Downloads folder, extract it as follows:
```bash
$ tar xf /mnt/c/Users/<user name>/Downloads/qbpp_amd64_<version>.tar.gz
```

This creates a directory named `qbpp_amd64_<version>` containing all required files.

It is recommended to create a symbolic link to this directory:
```bask
$ ln -s qbpp_amd64_<version> qbpp
```
This creates a symbolic link named `qbpp`, which simplifies access to the installation directory.

## Setting environment variable
Execute the following commands to set the environment variables required to compile and run QUBO++ programs:
```bash
$ export QBPP_PATH=$HOME/qbpp
$ export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
$ export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
$ export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
$ export PATH=$QBPP_PATH/bin:$PATH
```
If you have a QUBO++ license key, set it using:
```bash
$ export QBPP_LICENSE_KEY=[Your QUBO++ license key]
```
It is recommended to append these commands to the end of the `~/.bashrc` file so that they are automatically executed when the WSL shell starts.

## Activate license
You can now activate the QUBO++ license by executing:
```bash
$  qbpp-license -a
```
If a QUBO++ license key has been set, the corresponding license will be activated.
Otherwise, an anonymous license will be activated.

## Compile and execute the sample programs
Compile the QUBO++ sample programs using the following commands:
```bash
$ cd qbpp/samples
$ make
```
For example, you can run the QUBO++ program that solves the N-Queens problem as follows:
```bash
$ ./nqueen_easy
```

## Execute ABS3 GPU Solver
If your system has a CUDA-enabled GPU, the ABS3 GPU Solver can be executed from WSL.

To enable GPU acceleration in WSL, install the NVIDIA GPU driver for Windows from the following page:

https://www.nvidia.com/Download/index.aspx

> **NOTE**
> Do not install a Linux GPU driver inside WSL.
> WSL uses the Windows GPU driver via its integration layer.


After installing the driver, verify that the GPU is available in WSL by executing:
```bash
$ nvidia-smi
```
If the driver is installed correctly, this command displays information about the installed GPU.
You can then execute a sample program using the ABS3 GPU Solver as follows:
```bash
$ ./labs_abs3
```

## Upgrading to a new version of QUBO++
Download and extract the new QUBO++ release using `tar`, as described above.
Then update the qbpp symbolic link to point to the new version as follows:
```bash
$ ln -sfn qbpp_amd64_<new version> qbpp
```
This command overwrites the existing qbpp symbolic link so that it refers to the newly installed version.