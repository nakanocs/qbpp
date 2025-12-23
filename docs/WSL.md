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