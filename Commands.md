**Commands Used for Raspberry Pi GPIO LKM Project**

This file lists all the commands I used to set up, compile, and test the Linux Kernel Module (LKM) for controlling GPIO pins on a Raspberry Pi. These steps are designed to help you replicate my setup, including cross-compilation, kernel building, root filesystem creation, QEMU emulation, and module testing. The commands assume a Raspberry Pi (tested on Raspberry Pi 3B+) or a QEMU-based ARM environment running a compatible Linux distribution (e.g., Raspberry Pi OS or a custom setup).

**Prerequisites**

**Hardware:** Raspberry Pi (tested on 3B+) or a system for QEMU emulation.

**OS:** Raspberry Pi OS or a Linux system for cross-compilation (e.g., Ubuntu).

**Tools:** Cross-compilation toolchain, QEMU, Busybox, and kernel headers.

**GPIO Setup:** An LED or multimeter to verify GPIO output (optional for physical Raspberry Pi).

**Files:** Ensure lkm_gpio.c (LKM source) and Makefile are in the project directory.

**Step-by-Step Commands**

**1. Project Setup**

**Install necessary tools for cross-compilation, kernel building, and QEMU emulation:**

sudo apt update

sudo apt install -y build-essential gcc-arm-linux-gnueabihf qemu-system-arm libncurses-dev libssl-dev bc bison flex u-boot-tools

**2. Kernel Compilation**

Compile a custom Linux kernel for ARM (e.g., for Raspberry Pi or QEMU):

# Clone the Linux kernel source (adjust version as needed)
git clone --depth=1 --branch v6.14 https://github.com/torvalds/linux.git

cd linux

# Configure kernel for ARM
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- defconfig

# Build the kernel image
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- zImage -j$(nproc)

**3. Root Filesystem Creation**

Create a minimal root filesystem using Busybox:

# Download and extract Busybox (adjust version as needed)
wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2

tar -xjf busybox-1.37.0.tar.bz2

cd busybox-1.37.0

# Configure Busybox
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- defconfig

# Build and install Busybox
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- install

# Create compressed root filesystem
cd _install
find . | cpio -o --format=newc | gzip > ../../rootfs.cpio.gz
cd ../..

**4. QEMU Execution**

Test the kernel and root filesystem in QEMU’s ARM environment:

qemu-system-arm -M virt -m 256M \
-kernel linux/arch/arm/boot/zImage \
-initrd rootfs.cpio.gz \
-append "console=ttyAMA0 rdinit=/bin/sh" \
-nographic

**5. Compile the Kernel Module**

Build the LKM in the project directory (assuming lkm_gpio.c and Makefile are present):

cd /path/to/RPi-GPIO-LKM
make

This generates lkm_gpio.ko. Ensure the kernel headers match the kernel version used (e.g., 6.14).

**6. Kernel Module Management**

Load and test the kernel module (on Raspberry Pi or QEMU):

# Insert the module
sudo insmod lkm_gpio.ko

# Check kernel logs for initialization messages
dmesg | tail

# Remove the module
sudo rmmod lkm_gpio

# Verify removal
dmesg | tail

**7. Test GPIO Output**

If running on a physical Raspberry Pi, connect an LED (with a resistor) to the configured GPIO pin (e.g., GPIO 17) and ground. The pin should toggle based on the module’s logic. Use a multimeter to measure voltage if needed. In QEMU, GPIO testing may require additional setup (e.g., virtual GPIO emulation).

**8. Clean Up**

Remove compiled module files:

make clean

**Troubleshooting**

**Module Fails to Load:**

Verify kernel headers match the running kernel (uname -r on Raspberry Pi or QEMU kernel version).

Reinstall headers if needed: sudo apt install --reinstall raspberrypi-kernel-headers




**QEMU Errors:**

Ensure zImage and rootfs.cpio.gz paths are correct.

Check QEMU version compatibility (qemu-system-arm --version).


**No GPIO Output:**

Confirm the GPIO pin number in lkm_gpio.c matches your setup.

Check wiring for physical Raspberry Pi or emulate GPIO in QEMU.


**Kernel Crash:**

Review dmesg logs for errors.

Incorrect GPIO register access can cause crashes; verify base addresses in lkm_gpio.c.



**Notes**

**Environment:** Commands were tested on a Raspberry Pi 3B+ with Raspberry Pi OS and in a QEMU ARM environment (kernel 6.14, Busybox 1.37.0).

**Customization:** Adjust kernel version, Busybox version, or GPIO pin in lkm_gpio.c as needed.

**Safety:** Use sudo for kernel module operations and handle GPIO pins carefully to avoid hardware damage.

**QEMU:** Ideal for testing without physical hardware, but GPIO functionality may require additional configuration.

These commands reflect my workflow for developing and testing the LKM. For more context, read THEORY.md to understand the project’s design and limitations. Feel free to experiment and share your findings!
