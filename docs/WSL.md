---
layout: default
title: "Windows (WSL)"
nav_order: 1
parent: "Quick Start"
---
# Quick Start for Windows (WSL)
On Windows 11, QUBO++ can be used through **WSL (Windows Subsystem for Linux)**, which allows Linux programs to run natively on Windows.

This document explains how to install WSL, required libraries, and QUBO++, and how to compile and run sample programs.

## Install WSL
On Windows 11, **WSL** can be installed from Windows PowerShell.
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
Enter your **[user account name]** and **[your password]** when prompted.
This installs WSL 2 and an Ubuntu-based Linux system running on Windows.

After the installation is complete, update and upgrade the system software inside WSL:
```bash
sudo apt update
sudo apt upgrade -y
```

## Install C++ compiler
QUBO++ requires a **C++ compiler**.

Install it using the following command:
```bash
sudo apt install -y build-essential
```

## Install QUBO++

There are two ways to install QUBO++:
- **Method 1: apt (recommended)** — Simple installation with automatic path configuration
- **Method 2: tar.gz** — Manual installation without requiring apt repository setup

### Method 1: Install via apt (recommended)

First, add the QUBO++ apt repository:
```bash
curl -fsSL https://nakanocs.github.io/qbpp-apt/KEY.gpg | sudo gpg --dearmor -o /usr/share/keyrings/qbpp.gpg
echo "deb [signed-by=/usr/share/keyrings/qbpp.gpg] https://nakanocs.github.io/qbpp-apt stable main" | sudo tee /etc/apt/sources.list.d/qbpp.list
```

Then install QUBO++:
```bash
sudo apt update
sudo apt install qbpp
```

This automatically installs headers to `/usr/local/include/qbpp/`, shared libraries to `/usr/local/lib/`, and the `qbpp-license` command to `/usr/local/bin/`.
No environment variable configuration is needed.

The sample programs are installed to `/usr/local/share/qbpp/samples/`.
To compile and run them:
```bash
cp -r /usr/local/share/qbpp/samples ~/qbpp_samples
cd ~/qbpp_samples
make
./nqueen_easy
```

To upgrade to a new version:
```bash
sudo apt update
sudo apt install --only-upgrade qbpp
```

To uninstall:
```bash
sudo apt remove qbpp
```

### Method 2: Install via tar.gz

Download the `.tar.gz` file of the latest QUBO++ release from the [**Latest Release**](https://github.com/nakanocs/qbpp/releases/latest) page.

Download one of the following files, depending on your Windows PC architecture:
- **`qbpp_amd64_<version>.tar.gz`** : For Intel- or AMD-based Windows PCs
- **`qbpp_arm64_<version>.tar.gz`** : For ARM-based Windows PCs (e.g., Copilot+ PCs)


If the file is downloaded to your Windows Downloads folder, extract it as follows:
```bash
tar xf /mnt/c/Users/<user name>/Downloads/qbpp_<arch>_<version>.tar.gz
```

This creates a directory named **`qbpp_<arch>_<version>`** containing all required files.


It is recommended to create a symbolic link to this directory:
```bash
ln -s qbpp_<arch>_<version> qbpp
```
This creates a symbolic link named **`qbpp`**, which simplifies access to the installation directory.

#### Set environment variables
Execute the following commands to set the **environment variables** required to compile and run QUBO++ programs:
```bash
export QBPP_PATH=$HOME/qbpp
export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
export PATH=$QBPP_PATH/bin:$PATH
```
It is recommended to append these commands to the end of the **`~/.bashrc`** file so that they are automatically executed when the WSL shell starts.

#### Compile and run sample programs
```bash
cd qbpp/samples
make
./nqueen_easy
```

#### Upgrading to a new version
Download and extract the new QUBO++ release using `tar`, as described above.
Then update the qbpp symbolic link to point to the new version as follows:
```bash
ln -sfn qbpp_<arch>_<new version> qbpp
```
This command overwrites the existing qbpp symbolic link so that it refers to the newly installed version.

## Activate license
If you have a **QUBO++ license key**, set it using:
```bash
export QBPP_LICENSE_KEY=[Your QUBO++ license key]
```
It is recommended to append this command to the end of the **`~/.bashrc`** file.

You can then **activate the QUBO++ license** by executing:
```bash
qbpp-license -a
```
If a QUBO++ license key has been set, the corresponding license will be activated.
Otherwise, an anonymous license will be activated.

## Execute ABS3 GPU Solver
If your system has a **CUDA-enabled GPU**, the ABS3 GPU Solver can be executed from WSL.

To enable GPU acceleration in WSL, install the **NVIDIA GPU driver for Windows** from the following page:

https://www.nvidia.com/Download/index.aspx

> **NOTE**
> Do not install a Linux GPU driver inside WSL.
> WSL uses the Windows GPU driver via its integration layer.


After installing the driver, verify that the GPU is available in WSL by executing:
```bash
nvidia-smi
```
If the driver is installed correctly, this command displays information about the installed GPU.
You can then execute a sample program using the ABS3 GPU Solver as follows:
```bash
./labs_abs3
```
