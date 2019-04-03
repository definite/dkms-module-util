# UEFI Secure Boot DKMS Module Utilities

This provides an instruction and utilities for signing DKMS kernel modules
for UEFI secure boot. This instruction is mainly for Fedora and RHEL,
modify this to suit your environment.

So far, following modules are provided:

* evdi ([Displaylink](https://www.displaylink.com/))

# Steps

## 0. Register Your Signing Key Pairs

This section is derived from [Signing Kernel Modules for Secure Boot](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Administration_Guide/sect-signing-kernel-modules-for-secure-boot.html)

Steps in this section only need to be run once. Skip this part if your signing keys are enrolled in the system.

### 0.1 Clone or Download This Repository
```
git clone https://github.com/definite/dkms-modules
cd dkms-modules
```

### 0.2 Generating a Public and Private X.509 Key Pair
Firstly edit `openssl.cnf` and change the `CHANGEME`.

Then run:
```sh
openssl req -x509 -new -nodes -utf8 -sha256 -days 36500 -batch \ 
-config openssl.cnf -outform DER \
-out MOK.der -keyout MOK.priv
```

Move the keys to a secure directory. Assume it to be `/root`.

### 0.3 Enrolling Public Key on Target Machine
A Machine owner key (MOK) is a machine-owner-generated key to sign EFI binaries, such as kernel modules.
Previous section we have `MOK.der` the public key and `MOK.priv` the private key.

 1. Enroll the `MOK.der` for to UEFI with:
    ```sh
    sudo mokutil --import /root/MOK.der
    ```
    Remember the password, you need it in step 3.
 2. Reboot
 3. Pending MOK key enrollment invokes MokManager in UEFI console.
    You will need to enter password from step 1 to finish enrolling.

### 0.4 Edit `$HOME/.config/dkms-sign.conf`
The command dkms is most likely to be run as root, so assuming `$HOME` is `/root`:

Edit `/root/.config/dkms-sign.conf` with content like:
```
KEY_DIR=AbsoluteDirThatContainsBothMOKKeys
```

## 1. Prepare the Module Source
This section is only required when install the first time or module new version.
We use the evdi module as example.

### 1.1 Install the module source
Download and extract the source to `/usr/src`
For example, evdi-1.6.0 should be extracted as `/usr/src/evdi-1.6.0`

### 1.2 Provide `dkms.conf`
`dkms.conf` specify how to build dkms module. 
Use the `evdi-dkms.conf` as template and save to the module source directory as `dkms.conf`

For example, for evdi-1.6.0, the modified `dkms.conf` should be saved as `/usr/src/evdi-1.6.0/dkms.conf`

### 1.3 Provide `sign_module.sh`
`sign_module.sh` is a bash script to help signing modules.
This file should be copied to the module source directory as `sign_module.sh`.

`sign_module.sh` need to be executable. Ensure it by:
```sh
chmod 755 sign_module.sh
```

## 2. Sign And Install Kernel Module
dkms command should be working by now.

For example, to build,sign and install evdi-1.6.0 for current kernel:
```sh
sudo dkms install evdi/1.6.0 -k $(uname -r)
```

And to remove evdi/1.6.0 for current kernel:
```sh
sudo dkms remove evdi/1.6.0 -k $(uname -r)
```

Enjoy!

