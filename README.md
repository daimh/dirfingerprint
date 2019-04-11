usage: dirfingerprint [-h] [-x] [--gluster-brick GLUSTER_BRICK] [--version]

                      [DIR [DIR ...]]



It is always a pain in the eyes to compare two huge directories. Command 'du' doesn't work in all cases because it counts the size of each regular file, directory, soft-link, pipe, block device, etc, For example, if a directory has only empty files, 'du' reports a non-zero size for the directory, and the size varies depending on the underlying filesystem type. Most ordinary users actually don't care about those special files' size.



To rub salt in a wound, how can we find the folders that are renamed or moved to another location?



Further, almost all distributed filesystems don't have a good performance to access their metadata. Repeating some commands such as 'rsync -nav' on them isn't efficient.



Thus, here is 'dirfingerprint', it reports 6 properties for each subdirectories. The properties are calculated on the subdirectory recursively.

FileBytes:	total size of the regular files. This number should be smaller than a 'du -b' as it counts regular files only.

FileCount:	count of the regular files

FileMinMtime:	the oldest mtime of the regular files

FileMaxMtime:	the newest mtime of the regular files

SubdirCount:	count of all subdirectories

SpecialCount:	count of all special files that are not regular file or directory

FingerPrint:	a md5sum based on three parts. The first part is the two numbers of SubdirCount and SpecialCount, the second part is a list of name, size and mtime of the regular files that are one level under the directory, and the third part is a list of name and FingerPrint of all the subdirectories that are one level under the directory.

Dir:	name of the subdirectory



Here is an example of how FingerPrint is calculated, run these commands

$ mkdir test

$ touch -t "201901010101" test/f1

$ touch -t "201901010101" test/f2

$ mkdir test/subdir1

$ mkdir test/subdir1/subdir2

$ dirfingerprint test

#FingerPrint calculation of the empty directory 'test/subdir1/subdir2'

$ echo "0 0" | md5sum

#FingerPrint calculation of the directory 'test/subdir1'

$ echo -e "1 0\nsubdir2 5928dd99059f0c73963285d86f359fdb" | md5sum

#FingerPrint calculation of the directory 'test'

$ stat -c %Y test/f1 test/f2 #get the mtime

$ echo -e "1 0\nf1 0 1546322460\nf2 0 1546322460\nsubdir1 eecee4276b062aa6c1717dde01094d1d" | md5sum



An experimental feature is GlusterFS support, it can access GlusterFS brick nodes and get the metadata directly without network delay. an example is

$ dirfingerprint --gluster-brick=node1:/brick --gluster-brick=node2:/brick .



positional arguments:

  DIR                   directory



optional arguments:

  -h, --help            show this help message and exit

  -x, --one-file-system

                        skip directories on different file systems

  --gluster-brick GLUSTER_BRICK

                        HOSTNAME:PREFIX

  --version             output version information and exit

