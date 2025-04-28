# SmokelessRuntimeEFIPatcher

SmokelessRuntimeEFIPatcher (SREP) enables patching and injection of EFI modules at runtime. SREP serves as an alternative to SPI flashing, allowing BIOS/UEFI modifications without physical hardware access.

SREP capabilities include:
- Loading and patching EFI modules from the filesystem
- Loading modules from firmware volumes
- Executing pattern-based patches
- Launching EFI applications

SREP reads a configuration file (`SREP_Config.cfg`) from a USB drive and executes the specified operations. Users can create custom configurations to unlock hidden BIOS options, modify firmware behavior, or make other EFI-level changes without reflashing the BIOS chip.

See the usage instructions below for details on creating configuration files, supported operations, and build instructions.

## Getting Started

1. Disable Secure Boot and enable USB booting in your BIOS/UEFI settings before proceeding.
2. Create a bootable USB drive with a GPT partition scheme and FAT32 EFI System Partition
3. Copy the SREP application to `EFI/BOOT/BOOTX64.EFI` on the USB drive
4. Create a `SREP_Config.cfg` file in the root directory of the USB drive
5. Boot your system from the USB drive (may require pressing F12, ESC, or another key during startup to access boot menu)

## Configuration File Format

The configuration file consists of operation blocks. Each block starts with `Op` followed by an operation name and ends with `End`. Operations can have multiple arguments on separate lines:

    Op OpName1
        Argument 1
        Argument 2
        Argument n
    Op OpName2
        Argument 1
        Argument 2
        Argument n
    End

    Op OpName3
        Argument 1
        Argument 2
        Argument n
    End

### Supported Operations

SREP supports the following operations:

| Operation  | Arguments                                              |
| ---------- | ------------------------------------------------------ |
| LoadFromFS | `FileName`                                             |
| LoadFromFV | `SectionName`                                          |
| Loaded     | `Name`                                                 |
| Patch      | `Pattern`, `Offset`, `RelNegOffset`, or `RelPosOffset` |
| Exec       | None                                                   |

Description of operations:
- **LoadFromFS**: Loads an EFI file from the filesystem into memory and sets it as the current target for subsequent operations.
- **LoadFromFV**: Loads an EFI module from the Firmware Volume (BIOS image) into memory and sets it as the current target.
- **Loaded**: Targets an already loaded EFI module in memory for subsequent operations.
- **Patch**: Applies modifications to the currently targeted module in memory.
  - `Pattern`: Finds a byte pattern and replaces it with another pattern
  - `Offset`: Specifies an absolute offset from the start of the file and the bytes to replace
  - `RelNegOffset`/`RelPosOffset`: Specifies a negative/positive offset relative to the previous patch operation and the bytes to replace
- **Exec**: Executes the currently targeted EFI module.

## Examples

### Basic Example: Load and Execute an EFI Application

    Op LoadFromFS APP.efi
    Op Exec
    End

### Pattern Replacement Example

This example loads an EFI file, performs two pattern replacements, and executes it:

    Op LoadFromFS
    APP.efi
    Op Patch
    Pattern
    AABBCCDDEEFF
    AABBCCDDEEEE
    Op Patch
    Pattern
    AABBCCDDAABB
    AABBCCDDAAAA
    Op Exec
    End

### Relative Offset Example

Find the pattern AABBCCDDEEFF (replace with AABBCCDDEEFF, as we want it's own start address), then write AABBCCDDAAAA, at +50 from the pattern start

    Op LoadFromFS
    APP.efi
    Op Patch
    Pattern
    AABBCCDDEEFF
    AABBCCDDEEFF
    Op Patch
    RelPosOffset
    50
    AABBCCDDAAAA
    Op Exec
    End

## Build Instructions

This project is built and tested with Ubuntu 24.04 LTS. While it may work with other Linux distributions or versions, Ubuntu 24.04 LTS provides the most reliable environment for building the application with all dependencies properly resolved.

1. Install required dependencies:
   ```bash
   sudo apt install build-essential nasm uuid-dev
   ```

2. If Python is not found, create a symlink:
   ```bash
   sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1
   ```

3. Clone the EDK2 repository:
   ```bash
   git clone https://github.com/tianocore/edk2.git
   ```

4. Clone this repository inside the edk2 directory:
   ```bash
   cd edk2
   git clone https://github.com/TommyLau/SmokelessRuntimeEFIPatcher.git
   ```

5. Build the EDK2 BaseTools:
   ```bash
   make -C BaseTools
   ```

6. Set up the EDK2 environment:
   ```bash
   source edksetup.sh
   ```

7. Build the project:
   ```bash
   build -b RELEASE -t GCC5 -p SmokelessRuntimeEFIPatcher/SmokelessRuntimeEFIPatcher.dsc -a X64 -s
   ```

8. The compiled EFI application will be available at:
   ```
   Build/SmokelessRuntimeEFIPatcher/RELEASE_GCC5/X64/SmokelessRuntimeEFIPatcher.efi
   ```

9. Copy the EFI file to a USB drive in the `EFI/BOOT` directory as `BOOTX64.EFI` and create your SREP_Config.cfg file in the root of the USB drive.

## Credits
Originally created by [SmokelessCPUv2](https://github.com/SmokelessCPUv2/)
