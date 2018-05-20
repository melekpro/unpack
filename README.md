# unpack
allows you to work with Linux Kernel zImage (ARM, Little Endian) from MTK (Mediatek). Suppose you dragged a Linux Kernel image from boot.img or recovery.img. In general, it consists of the kernel binary itself, which is compressed by gzip, dtb, loader, and possibly some service information.

There are several options, for example:

    At the command file recovery.img-kernel, the content is defined as Linux kernel ARM boot executable zImage (little-endian).
    Content is defined as gzip compressed data, max compression, from Unix.

In the first case, the file structure is the following, first the code is boot executable, then the GZip of the kernel itself, then the offset table and then the DTB, starting with the signature DTB_MAGIC - D0 0D FE ED (0xEDFE0DD0). What is the displacement table?

    DWORD (0x0)
    DWORD (0x0)
    DWORD (0x0)
    Pointer to DTB_MAGIC
    A pointer to the offset table (that is, the first DWORD from this list)
    ... (etc.)

In the second case, the structure is simpler, no boot executable is available, just go GZip kernel, and then DTB, starting with dtb-magic.

The script correctly processes (at least on test cases) both versions.

The launch is performed as follows:

unpack.sh recovery.img-kernel

As a result of the script, three files will be generated:

    1_kernel_header.bin
    2_kernel_gzip.gz
    3_kernel_footer.bin

With the names, I think everything is clear - in the first there is the code boot executable (if there is one), in the second one directly pure kernel.gz, and in the third the table of displacements (if any) + DTB. The total size of these three files must be equal to the size of the original recovery.img-kernel. If this condition is met, then the kernel is decompressed correctly.

Now you can unzip 2_kernel_gzip.gz and learn (or modify) the kernel. When modifying, it is important that the size of the resulting gz is equal to the size of the original gzip. You can collect everything back like this:

cat 1_kernel_header.bin 2_kernel_gzip.gz 3_kernel_footer.bin> recovery.img-kernel-new

The script was written thanks to the questions that arose when communicating with jemmini and hyperion70. On perfection in any case does not pretend, just a convenient "wholes" to not run that HIEW from under Wine and do not cut recovery.img-kernel manually.
