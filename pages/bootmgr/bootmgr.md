_The text on this page is copied almost verbatim from parts of the UEFI Specification Version 2.3.1._

#Firmware Boot Manager

The boot manager is a component in UEFI firmware that determines which drivers and applications should be explicitly loaded and when. When the boot manager receives control, it is responsible for determining what to load and any interactions with the user necessary to make that decision.

## Load Options and Boot Order

Programmatic interaction with the boot manager is accomplished through globally defined variables. On initialization the boot manager reads the values which comprise all of the published load options among the UEFI environment variables.

Each load option entry resides in a `Boot####` variable or a `Driver####` variable where the `####` is replaced by a unique option number in printable hexadecimal representation using the digits `0–9`, and the upper case versions of the characters `A–F` (`0000–FFFF`).  The `####` must always be four digits, so small numbers must use leading zeros. 

The load options are then logically ordered by an array of option numbers listed in the desired order. There are two such option ordering lists. The first is `DriverOrder` that orders the `Driver####` load option variables into their load order. The second is `BootOrder` that orders the `Boot####` load options variables into their load order.

For example, to add a new boot option, a new `Boot####` variable would be added. Then the option number of the new `Boot####` variable would be added to the `BootOrder` ordered list and the BootOrder variable would be rewritten. To change boot option on an existing `Boot####`, only the `Boot####` variable would need to be rewritten. A similar operation would be done to add, remove, or modify the driver load list.

The boot manager is required to process the Driver load option entries before the Boot load option entries. The boot manager is also required to initiate a boot of the boot option specified by the `BootNext` variable as the first boot option on the next boot, and only on the next boot. The boot manager removes the `BootNext` variable before transferring control to the `BootNext` boot option. After the `BootNext` boot option is tried, the normal `BootOrder` list is used. To prevent loops, the boot manager deletes this variable before transferring control to the preselected boot option.

## Boot Behavior is Dynamic (not stable)

The boot manager may perform automatic maintenance of the database variables. For example, it may remove unreferenced load option variables or any load option variables that cannot be parsed or loaded, and it may rewrite any ordered list to remove any load options that do not have corresponding load option variables. In addition, the boot manager may automatically update any ordered list to place any of its own load options where it desires. The boot manager can also, at its own discretion, provide for manual maintenance operations as well. Examples include choosing the order of any or all load options, activating or deactivating load options, etc.

## Booting with Load Options

When booting with a load option, the boot manager must call `LoadImage()` (from boot-service table) which supports at least `EFI_SIMPLE_FILE_SYSTEM_PROTOCOL` and `EFI_LOAD_FILE_PROTOCOL` for resolving load options.  If `LoadImage()` succeeds, the boot manager must enable the watchdog timer for 5 minutes by using the `SetWatchdogTimer()` boot service prior to calling `StartImage()`. If a boot option returns control to the boot manager, the boot manager must disable the watchdog timer with an additional call to the `SetWatchdogTimer()` boot service.

If the boot via `Boot####` returns with a status of `EFI_SUCCESS`, the boot manager will stop processing the `BootOrder` variable and present a boot manager menu to the user. If a boot via `Boot####` returns a status other than `EFI_SUCCESS`, the boot has failed and the next `Boot####` in the `BootOrder` variable will be tried until all possibilities are exhausted.

Further details of boot order can be found in UEFI Specification 2.3.1, Section 3.1.2.

## Boot Manager Programming

By using the `SetVariable()` function the data that contain these environment variables can be modified. Such modifications are guaranteed to take effect after the next system boot commences.

These `BOOT####` variables contain a variable-length structure called an `EFI_LOAD_OPTION`. Since some of the fields are variable length, an `EFI_LOAD_OPTION` cannot be described as a standard C data structure.  Instead, the fields are listed below in the order that they appear in an `EFI_LOAD_OPTION` descriptor: 

        UINT32                      Attributes;
        UINT16                      FilePathListLength;
        CHAR16                      Description[];
        EFI_DEVICE_PATH_PROTOCOL    FilePathList[];
        UINT8                       OptionalData[];

Detalis of this structure can be found in the UEFI Specification 2.3.1, Section 3.1.3.

## Default Behavior (no working Load Options)

If no valid boot options exist, the boot manager will enumerate all removable media devices followed by all fixed media devices. The order within each group is undefined. These new default boot options are not saved to non volatile storage. The boot manger will then attempt to boot from each boot option. 

If the device supports the `EFI_SIMPLE_FILE_SYSTEM_PROTOCOL` (is FAT formatted) then the removable media boot behavior is executed. On a removable media device it is not possible for the `FilePath` to contain a file name, including sub directories. The load option is stored in non volatile memory in the platform and cannot possibly be kept in sync with a media that can change at any time. The `FilePathList[0]` field in the load option for a removable media device will point to a device. The `FilePathList[0]` will not contain a file name or sub directories.

On an appropriately formatted device, the system firmware will attempt to boot using the file path: `\EFI\BOOT\BOOT{`machine type short-name`}.EFI`. Machine type short-name defines a `PE32+` image format architecture.  Each file only contains one UEFI image type, and a system may support booting from one or more images types. A media may support multiple architectures by simply having a `\EFI\BOOT\BOOT{`machine type short-name`}.EFI` file of each possible machine type.

On x86-64 platforms, the default file path is: `\EFI\BOOT\BOOTx64.EFI`.

## Note on Security

The `Boot####` and `BootOrder` variables are not authenticated. As a result, access to Run-Time services are sufficient to create, delete, or alter the load options. Note: the load options include the "bootline" passed to the EFI application. As a result, even if secure boot is enabled and the integrity of EFI application images is protected, the integrity of the boot line is not protected.

