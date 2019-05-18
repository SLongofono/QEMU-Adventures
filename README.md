# QEMU-Adventures
A living documentation of things to do with QEMU.  [Thanks to A. Holbreich for the quickstart.](http://alexander.holbreich.org/qemu-kvm-introduction/)

## Installation
Step 0: Do we support HW virtualization, and is it enabled?  If nonzero, we are good.
```
grep -c '(vmx|cmx)' /proc/cpuinfo
```

Check if enabled, and if not, make it so...
```
lsmod | grep kvm

modprobe kvm && sudo modprobe kvm_intel
```
Install and set up user access
```
sudo apt-get install qemu-kvm
sudo adduser $USER kvm
```

## Setting Up Images for VM Persistence
There are a variety of formats, all of which will be transparent to the guest OS.  QEMU will essentially trap instructions to a specific address space and handle how they map to the address space in the image of choice.
### Image Types
+ **raw** - default format. Allows flexible converting. Takes only used side on host, but only if used ext4 and (ext3?). This is called spare file or spare image

+ **qed** - enhanced disc format for faster access (since QEMU 0.14). It supports overlay and sparse images. Overlay means that on create you can assign allready existing image as base and only differences will be written to the overlay image (Beware: base image has to stay unchanged!). It's also faster than qcow2.

+ **qcow2** - most featured format in QEMU and is meant to replace qcow. This format support sparse images independent of underlying fs capabilities. It supports multiple VM-snapshots, encryption (AES) and compression.

+ **qcow** - old QEMU format. Images in qcow are sparse and like qcow2 independent of underlying file system capabilities.

+ **vmdk** - standard format for VMware Workstation. Overlay function is similar to qcow2.

+ **vdi** - standard format for Virtual Box.
parallels standard image tp of virtualization solutions of Paralles Inc.

+ **vpc** standard image format for Microsoft Virtual PC

### Creating Images
```shell
qemu-img create test.img 10G                            # Create a 10 GB image
qemu-img create -f qcow2 test.qcow2 1G                  # Create a 1 GB image in qcow2 format
qemu-img create -b test.img -f qcow2 testoverlay.ovl    # Create an overlay in the qcow2 format
qemu-img info test.img                                  # Inspect our handiwork
qemu-img resize test.img +5GB                           # Need more space plz
qemu-img convert -f qcow2 -O vdi input.img output.img   # Conversion is simple
qemu-img convert -f qcow2 -O qcow2 big.img smol.img     # Converting will find & drop unused space
qemu-img convert -c -O qcow2 source.img dest.img        # Compress with convert tool
qemu-img convert -O qcow2 -o encryption src.img dst.img # Secret stuff, prompts for a PW
```

## Launching
After creating an image and identifying a boot media (.iso file for example...), a single line will spin up a virtual machine instance.  Out of the box, a number of architectures are supported, and more can be added or implemented as needed.  Here, we use Alpine Linux since it is small and designed for VM use, and indicate that the VM should share the user's access to the internet via a VPN and NAT set up locally on the host:
```shell
qemu-system-x86_64 -hda disk.img -cdrom alpine-virt-3.9.4-x86_64.iso -net nic -net user
```
