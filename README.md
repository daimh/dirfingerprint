#### Install
```
 $ wget https://raw.githubusercontent.com/daimh/dirfingerprint/master/dirfingerprint
 $ chmod +x dirfingerprint
```
#### Example
* generate a tab delimited file for all directories under /tmp
```
$ dirfingerprint hash /tmp | tee 0.dfp
```
* find new/moved directory
```
$ mkdir /tmp/1 /tmp/2
$ dirfingerprint hash /tmp > 1.dfp
$ dirfingerprint diff 0.dfp 1.dfp
$ mv /tmp/1 /tmp/2
$ dirfingerprint hash /tmp > 2.dfp
$ dirfingerprint diff 1.dfp 2.dfp
```
* print information for all directories that have a depth of 2 or less
```
$ awk '$8 < 2 || NR == 1' 0.dfp
```
* sort all level-3 subdirectories by their size
```
$ awk '$8 == 3' 0.dfp | sort -k 2n
```
* GlusterFS support, access GlusterFS brick nodes and get the metadata directly without network delay

```
$ dirfingerprint hash --gluster-brick=node1:/brick --gluster-brick=node2:/brick .
```

#### Detail

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
```
$ mkdir test
$ touch -t "201901010101" test/f1
$ touch -t "201901010101" test/f2
$ mkdir test/subdir1
$ mkdir test/subdir1/subdir2
$ dirfingerprint hash test
## FingerPrint calculation of the empty directory 'test/subdir1/subdir2'
$ echo "0 0" | md5sum
## FingerPrint calculation of the directory 'test/subdir1'
$ echo -e "1 0\nsubdir2 5928dd99059f0c73963285d86f359fdb" | md5sum
## FingerPrint calculation of the directory 'test'
$ stat -c %Y test/f1 test/f2 #get the mtime
$ echo -e "1 0\nf1 0 1546322460\nf2 0 1546322460\nsubdir1 eecee4276b062aa6c1717dde01094d1d" | md5sum
```

## Copyright

Developed by [Manhong Dai](mailto:daimh@umich.edu)

Copyright © 2020 University of Michigan. License [GPLv3+](https://gnu.org/licenses/gpl.html): GNU GPL version 3 or later 

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.

## Acknowledgment

Ruth Freedman, MPH, former administrator of MNI, UMICH

Fan Meng, Ph.D., Research Associate Professor, Psychiatry, UMICH

Huda Akil, Ph.D., Director of MNI, UMICH

Stanley J. Watson, M.D., Ph.D., Director of MNI, UMICH
