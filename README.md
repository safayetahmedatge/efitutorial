# UEFI Tutorial

This is a tutorial to help developers ramp up on UEFI environment and programming. The intended audience is a C programmer with basic knowledge of computer architecture and system programming (paging, adressing modes ... etc).

## Useful Resources

Some useful resources are listed below. This tutorial copies a lot of material from the UEFI specification.

1. [UEFI Specification](http://www.uefi.org/specifications) (this tutorial uses Version 2.3.1)

2. [http://www.rodsbooks.com/](http://www.rodsbooks.com/efi-programming/index.html) (a little out of date)

3. [EFI Toolkit repo](https://github.com/tianocore/tianocore.github.io/wiki/EFI-Toolkit)

4. [EDKII documentation](https://github.com/tianocore/tianocore.github.io/wiki/EDKII-EADK)

5. [EDKII main repo](https://github.com/tianocore/edk2)

## What is UEFI?

The Unified Extensible Firmware Interface (UEFI) Specification describes an interface between the operating system (OS) and the platform firmware. UEFI is a replacement of the legacy BIOS interface.

> The interface is in the form of data tables that contain platform-related information, and boot and runtime service calls that are available to the OS loader and the OS. Together, these provide a standard environment for booting an OS.

Services provided by UEFI may be as simple as memory allocation and management or providing memory lay-out information, or launching another piece of UEFI code from a file. Also, UEFI supports more sophisticated services such as reading from or writing to FAT file systems, providing access to mouse, keyboard, and frame buffers, and even networking operations and security services. While the specification lists a large number of services, the actual firmware implementation may only provide a subset of those services.

## UEFI Images

UEFI executable images are packed as PE/COFF images (Windows binary format). Images may be loaded from a file system or over the network. Images may be executed directly by the firmware or by other UEFI executables through the firmware.

UEFI Images may be boot-service drivers or run-time-service drivers that implement boot-time or run-time services. However, the scope of this tutorial is limited to UEFI applications.

UEFI applications may be like utilities that perform a specific task and return control to the caller or launch another UEFI application (just like command-line utilities in Linux). However, unlike such applications, OS-loaders are a special type of application that take control of the system (away from UEFI boot-time services) and pass control to an operating system.

The concept of run-time services, boot-time services, and exit-boot-services are described in greater detail later in this tutorial. The following figure provides a summary of the different types of images.

![UEFI image types](pages/page0/uefi-image-types.jpg)

## Booting a UEFI Platform

UEFI platforms are expected to have a single EFI System Partition (ESP). The ESP is a specially marked FAT-formatted partition in a storage device (disk, USB flash drive, etc ...). The device may be partitioned using MBR or GPT. The firmware loads UEFI images from the EFI system partition.

UEFI platforms boot under the control of the UEFI Boot Manager in the firmware. The UEFI Boot Manager loads or attempts to load a sequence of drivers and applications based on some policy specified through UEFI variables. UEFI variables and boot policy are described in greater detail later in this tutorial.

If a valid image is not found through the boot policy, or in case of booting from removable media, the boot manager will attempt to load an image from a default path. The default path is architecture specific (x86, x86-64, Itanium, ARM). For x86-64 platforms, the default path is `\EFI\BOOT\BOOTx64.efi`. Since the ESP is FAT-formatted, file paths are not case sensitive.

