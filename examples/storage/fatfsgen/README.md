# FATFS partition generation on build example

(See the README.md file in the upper level 'examples' directory for more information about examples.)

This example demonstrates how to use the FATFS partition
generation tool [fatfsgen.py](../../../components/fatfs/fatfsgen.py) to automatically create a FATFS
filesystem image (without wear levelling support)
from the contents of a host folder during build, with an option of
automatically flashing the created image on invocation of `idf.py -p PORT flash`.
Beware that the minimal required size of the flash is 4 MB.
The generated partition does not support wear levelling,
so it can be mounted only in read-only mode.

The following gives an overview of the example:

1. There is a directory `fatfs_image` from which the FATFS filesystem image will be created.

2. The function `fatfs_create_partition_image` is used to specify that a FATFS image
should be created during build for the `storage` partition. For CMake, it is called from [the main component's CMakeLists.txt](./main/CMakeLists.txt). 
`FLASH_IN_PROJECT` specifies that the created image
should be flashed on invocation of `idf.py -p PORT flash` together with app, bootloader, partition table, etc.
The image is created on the example's build directory with the output filename `storage.bin`.

3. Upon invocation of `idf.py -p PORT flash monitor`, application loads and
finds there is already a valid FATFS filesystem in the `storage` partition with files same as those in `fatfs_image` directory. The application is then
able to read those files.

## How to use example

### Build and flash

To run the example, type the following command:

```CMake
# CMake
idf.py -p PORT flash monitor
```

(To exit the serial monitor, type ``Ctrl-]``.)

See the Getting Started Guide for full steps to configure and use ESP-IDF to build projects.

## Example output

Here is the example's console output:

```
...
I (322) example: Mounting FAT filesystem
I (332) example: Reading file
I (332) example: Read from file: 'this is test'
I (332) example: Unmounting FAT filesystem
I (342) example: Done
```

The logic of the example is contained in a [single source file](./main/fatfsgen_example_main.c), and it should be relatively simple to match points in its execution with the log outputs above.
