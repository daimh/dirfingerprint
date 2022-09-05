# Dirfingerprint

<details>
<summary>Table of contents</summary>
<ol>
    <li><a href='#about'>About</a></li>
    <li><a href='#install'>Install</a></li>
    <li><a href='#example'>Example</a></li>
    <li><a href='#license'>License</a></li>
    <li><a href='#acknowledgment'>Acknowledgment</a></li>
<ol>
</details>

## About
It is always a pain in the eyes to compare two huge directories. Command 'du' doesn't work in all cases because it also counts the metadata size, For example, if a directory has only empty files, 'du' reports a non-zero size for the directory and the size varies depending on the underlying filesystem type. Most ordinary users don't care about size of any non-files. Further, almost all distributed filesystems don't have a good performance to access their metadata. Repeating some commands such as 'rsync -nav' on them isn't efficient. To rub salt in a wound, how can we find the directories that were renamed or moved to another location?

Thus, here is 'dirfingerprint', it reports the following properties for each subdirectory. The properties are calculated on the subdirectory recursively.

FingerPrint:	a md5sum based on three parts. The first part is the two numbers of SubdirCount and SpecialCount, the second part is a list of name, size, and mtime of the regular files that are one level under the directory, and the third part is a list of name and FingerPrint of all the subdirectories that are one level under the directory.

FileBytes:	total size of the regular files. This number should be smaller than a 'du -b' as it counts regular files only.

FileCount:	count of the regular files

FileMinMtime:	the oldest mtime of the regular files

FileMaxMtime:	the newest mtime of the regular files

SubdirCount:	count of all subdirectories

SpecialCount:	count of all special files that are not regular file or directory
Depth:	depth of the sub-directory, 0 is the root directory

Dir:	name of the subdirectory

## Install
```
 $ wget https://raw.githubusercontent.com/daimh/dirfingerprint/master/dirfingerprint
 $ chmod +x dirfingerprint
 $ ./dirfingerprint -h
```
## Example
* generate a hash for all subdirectories under /tmp
```
dirfingerprint hash /tmp | tee 0.dfp
```
* find new/moved directory
```
mkdir /tmp/1 /tmp/2
dirfingerprint hash /tmp > 1.dfp
dirfingerprint diff 0.dfp 1.dfp
mv /tmp/1 /tmp/2
dirfingerprint hash /tmp > 2.dfp
dirfingerprint diff 1.dfp 2.dfp
dirfingerprint diff 0.dfp 2.dfp
```
* print information for all directories that have a depth of 2 or less
```
awk '$8 < 2' 0.dfp
```
* sort all level-3 subdirectories by their size
```
awk '$8 == 2' 2.dfp | sort -k 2n
```
* GlusterFS support, access GlusterFS brick nodes and get the metadata directly without the overhead caused by fuse

```
dirfingerprint hash --gluster-brick=node1:/brick --gluster-brick=node2:/brick .
```

## License

Developed by [Manhong Dai](mailto:daimh@umich.edu)

Copyright © 2022 University of Michigan. License [GPLv3+](https://gnu.org/licenses/gpl.html): GNU GPL version 3 or later 

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.

## Acknowledgment

Ruth Freedman, MPH, former administrator of MNI, UMICH

Fan Meng, Ph.D., Research Associate Professor, Psychiatry, UMICH

Huda Akil, Ph.D., Director of MNI, UMICH

Stanley J. Watson, M.D., Ph.D., Director of MNI, UMICH
