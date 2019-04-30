# Lab10: File Systems

In this lab assignment you will get experience working with file systems: creating a new FS, manipulating files, and inspecting the data organized on a disk.

## The Disk

As usual, everything on a Unix system is a file. The first thing we should do is find out what 'file' represents our main (boot) disk. You can use the `df` command to print out all the currently-mounted file systems, including their sizes and space available (use the `-H` option for human-readable sizes).

**(1.)**-a What device file represents your root (`/`) partition?

**(1.)**-b List the partition table type, file system type, and the disk's UUID. To do this, run:

```
udevadm info --query=all --name=/dev/DEVICE
```

Replace `DEVICE` with the device that hosts your root partition.

**(2.)** What is the output of `ls -l` on this file? What is different about this file compared to most?

Since disks are just files, we can create a disk by creating a file. (Yo dawg, I heard you like disks...):

```
dd if=/dev/zero of=./my-disk bs=1048576 count=1000
```

**(3.)** How big is this file, in megabytes? How long did it take to write this file to your VM's disk?

Okay, so we created a blank file (filled with zeros from /dev/zero -- which is also a device file, by the way). Now we can put a file system on it:

```
mkfs.ext4 ./my-disk
```

**(4.)** How many inodes are created by this process?


## Mounting a File System

The act of *mounting* a file system makes the data on a disk available to us under a particular *mount point*. A mount point is just a directory -- it doesn't have to be empty, but mounting a disk to a particular directory will hide (replace) whatever contents were there already. So most of the time, we'll mount our file systems to an empty directory.

In order to mount file systems, you must be the root user. Switch to root, and then:

```
mkdir /tmp/mount
touch /tmp/mount/test-file
ls -l /tmp/mount/
mount ~yourusername/my-disk /tmp/mount
ls -l /tmp/mount/
```

**(5.)** What are the contents of /tmp/mount (shown by `ls -l`) before and after you mount the disk?

**(6.)** How much space was used by creating the file system? In other words, how much free space does your disk have right now? You know the command to print this information out...

Make a directory in this new file system for your regular user account (call it 'my_files'). You will need to update its permissions with `chown` to make it accessible. Afterward, log out of the root account and continue as a regular user.

**(7.)** What is the output of `ls -ld /tmp/mount/my_files`? Are the permissions set up correctly?

## Manipulating The File System

Back as your regular user, cd to /tmp/mount/my_files. Let's create a couple files here:

```
touch fileA
umask 000
touch fileB
umask 777
touch fileC
umask 077
whoami | md5sum > whoami.txt
```

**(8.)** What is the output of `ls -l`? What difference does the `umask` command make to the file permissions?

**(9.)** What `umask` command would set the default permissions for files to `------r--` ?

**(10.)** What inode number is associated with fileB? (`ls -li`)

Okay, let's create a couple more files here. First, let's generate a file with some random data:

```
dd if=/dev/random of=./random-file bs=1024 count=1
```

**(11.)** What is the inode number of random-file?

We can also create a file with a nice greeting:

```
echo "hello world" > hello-file
```

**(12.)** After doing this, what happens when you run `grep -a hello ~/my-disk`? What's happening here?

Speaking of the file that represents our disk, let's inspect it a bit closer:

```
debugfs ~/my-disk
stat <number>
```

Where 'number' is the inode number of fileB and random-file.

**(13.)** What is the output of these two `stat` commands?

debugfs supports several commands to view file system information. Use `help` or `?` to show these commands.

**(14.)** Paste the superblock information for your FS here.

**(15.)** What command would you use to delete a directory via debugfs?

Okay, we're all done here. cd out of your file system, switch to root, and then un-mount it with:

```
umount /tmp/mount
```

Then you'll bundle your file system up and turn it in (seriously). To do this, compress it first:

```
gzip -9 my-disk
```

Then check in my-disk.gz to GitHub.

## Stat

In the last part of this assignment, you'll develop a simple C program that prints file information using the `stat` system call. (see `man 2 stat` for more information). Given a file path, stat will populate a struct with inode information.

The struct:

```

struct stat {
	dev_t     st_dev;         /* ID of device containing file */
	ino_t     st_ino;         /* Inode number */
	mode_t    st_mode;        /* File type and mode */
	nlink_t   st_nlink;       /* Number of hard links */
	uid_t     st_uid;         /* User ID of owner */
	gid_t     st_gid;         /* Group ID of owner */
	dev_t     st_rdev;        /* Device ID (if special file) */
	off_t     st_size;        /* Total size, in bytes */
	blksize_t st_blksize;     /* Block size for filesystem I/O */
	blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

	/* Since Linux 2.6, the kernel supports nanosecond
	   precision for the following timestamp fields.
	   For the details before Linux 2.6, see NOTES. */

	struct timespec st_atim;  /* Time of last access */
	struct timespec st_mtim;  /* Time of last modification */
	struct timespec st_ctim;  /* Time of last status change */

	#define st_atime st_atim.tv_sec      /* Backward compatibility */
	#define st_mtime st_mtim.tv_sec
	#define st_ctime st_ctim.tv_sec
};

```

So we can do something like:

```
    struct stat statbuf = { 0 };
    if (stat(path, &statbuf) == -1) {
        perror("stat");
        return;
    }
```

Your program will accept one command-line argument: the file path. It will then determine whether the file is owned by the same user running the process. So if a file is owned by 'mmalensek' and the user running your program is also 'mmalensek' it will print:

```
This file is owned by the current user
```

Otherwise,

```
This file is owned by someone else
```

In this case (not owned by the process), you will need to find out whether or not we can access the file (either we are in the same group with read permission, or the last (others) field has read permission). So:

```
This file is owned by someone else but *IS* readable by this process
```

or

```
This file is owned by someone else and is *NOT* readable by this process
```

Check the code in as inode_reader.c.


# Summing Up

Make sure you answered each question, as well as checked in:

* inode_reader.c
* my-disk.gz
