# Archived
This repo is now superseded by https://github.com/sev-step/sev-step

# What is this

This is a fork of the official [AMDSEV repo](https://github.com/AMDESE/AMDSEV) which also applies the kernel patch required for the sev step
framework. You can find the patch in `sev-step.patch`

The intention of this repo is to provide a convenient way to setup
an environment for the sev-step framework

# Build
We require three components: Qemu, OVMF and the Linux Kernel.
`./build.sh` builds all components.
As compiling the Linux kernel takes some time, we provide precompiled
binaries in the release section of this repo. In this case
issue `./build.sh ovmf` and `./build.sh qemu` to only build the remaining
components.

# Setup Host

1) Issue `sudo cp kvm.conf /etc/modprobe.d/` to ensure the KVM module loads
with the corrects params.

2) Install the "snp-host" kernel on the host system with
`sudo dpkg -i ./linux/linux-*snp-host*.deb` and boot into this kernel.

3) Verify that the following BIOS settings are enabled. The setting may vary based on the vendor BIOS. The menu option below are from AMD BIOS. 
    ```
    CBS -> CPU Common ->
                    SEV-ES ASID space Limit Control -> Manual
                    SEV-ES ASID space limit -> 100
                    SNP Memory Coverage -> Enabled 
                    SMEE -> Enabled
        -> NBIO common ->
                    SEV-SNP -> Enabled
    ```


# Setup VM

1) Download an Ubuntu image
2) Create a virtual disk with `qemu-img create -f qcow2 disk.qcow2 20G`
3) Install OS in VM by starting it with `./launch-qemu.sh -hda <path to disk file> -cdrom <path to ubuntu iso> -vnc :1`. Then connect via VNC to port `5901` (passing host:d to -vnc in qemu opens the server on host:5900+d, don't ask why) and follow the installation instructions
4) Start the VM again without the `-cdrom` arg and install the OpenSSH server. You can access port `22` on the VM via  `localhost:2222` on the host sytem.
5) Copy the "*snp-guest*" packages from the `linux/` folder into the VM via `scp -P222 linux/linux*-snp-guest*.deb`and install them (inside the VM) with `dpkg -i inux*-snp-guest*.deb`. Make sure the VM defaults to booting this kernel.
6) You can now start the VM with SNP activated via `./launch-qemu.sh -hda <path to disk file> -sev-snp`. Connect via ssh and issue `dmesg | grep "AMD Memory Encryption Features active"`. It should show `SEV SEV-ES SEV-SNP`.

# Use Attack Framework
Head over to https://github.com/UzL-ITS/sev-step to install the user space part of the attack framework.
