# A simple filesystem to understand things

This is a Work In Progress. Do not use this yet.

If you are planning to learn filesystems, start from the scratch. You can look from the first commit in this repository and move the way up.

The source files are licensed under Creative Commons Zero License.
* More information at:	http://creativecommons.org/publicdomain/zero/1.0/
* Full license text at:	http://creativecommons.org/publicdomain/zero/1.0/legalcode

## simplefs 1.0 Architecture + Notes

* Block Zero = Super block
* Block One = Inode store
* Block Two = Occupied by the initial file that is created as part of the mkfs.

Only a limited number of filesystem objects are supported.
Files and Directories can be created. Support for .create and .mkdir is implemented. Nested directories can be created.
A file cannot grow beyond one block. `ENOSPC` will be returned as an error on attempting to do.
Directories store the children inode number and name in their data blocks.
Read support is implemented.
Basic write support is implemented. Writes may not succeed if done in an offset. Works when you overwrite the entire block.
Locks are not well thought-out. The current locking scheme works but needs more analysis + code reviews.
Memory leaks may (will ?) exist.

## Credits

The initial source code is written by psankar and patches are contributed by other github members. azat being the first contributor.

An `O_LARGETHANKS` to the guidance of VijaiBabu M and Santosh Venugopal. Their excellent talks on filesystems motivated me to implement a file system from the scratch. Without their inspirational speeches, I would not have focussed on filesystems.

A big thanks should go to the kernelnewbies mailing list for helping me with clarifying the nitty-gritties of the linux kernel, especially people like Mulyadi Santosa, Valdis Kletnieks, Manish Katiyar, Rajat Sharma etc.

Special thanks to Ankit Jain who provides interesting conversations to help my intellectual curiosities.


## Patch Submission

Please send a merge request only if you are ready to publish your changes in the same creative commons zero license.

We would like to keep the filesystem simple and minimal so that it can be used as a good teaching material.

We are not planning to use this filesystem in a production machine. So some design choices are driven by learning/teaching simplicity over performance/scalability. This is intentional. So do not try to fix those things :)

## TODO

- After the 1.0 release, start with support for extents, which on completion will be 2.0
- After the 2.0 release, start with journalling support, which on completion will be 3.0

## Prerequisite

Install linux kernel header in advance.
```shell
$ sudo apt install linux-headers-$(uname -r)
```

## Build and Test

```shell
$ mkdir -p mount # the mount point
$ make
$ dd bs=4096 count=100 if=/dev/zero of=image
$ ./mkfs-simplefs image
```

You shall get the following messages:
```
Super block written succesfully
root directory inode written succesfully
journal inode written succesfully
welcomefile inode written succesfully
inode store padding bytes (after the three inodes) written sucessfully
Journal written successfully
root directory datablocks (name+inode_no pair for welcomefile) written succesfully
padding after the rootdirectory children written succesfully
block has been written succesfully
```

Let's test kernel module:
```shell
$ sudo insmod simplefs.ko
$ sudo mount -o loop -t simplefs image `pwd`/mount/
$ dmesg
$ ls mount/
$ cat mount/vanakkam
```

You shall get the message:
```
Love is God. God is Love. Anbe Murugan.
````

Then, you can perform regular file system operations:
```shell
$ echo "Hello World" > mount/hello
$ cat mount/hello
$ ls -lR 
```

Remove kernel mount point and module:
```shell
$ sudo umount mount
$ sudo rmmod simplefs
```
