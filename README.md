# APFS Japanese Kana Compatibility PoC

Read more on this detailed post: [Accented Latin and Kana File Names Finally Work as Expected in APFS](https://medium.com/@yorkxin/apfs-docker-unicode-6e9893c9385d)

## Problem of Kana File Name

In HFS+, file name will be decomposed to Unicode NFD form. So a file named `ありがとう` will be decomposed to `ありか<U+3099>とう`,
where [U+3099 is a 濁点 sound mark](https://codepoints.net/U+3099).

When we build an image that is based on Linux, Docker doesn't compose the string back to NFC form, 
thus any file with 濁点 or 半濁点 will not be accessible.

For example:

```
$ docker build -t test-image .
$ docker run test-image cat ありがとう.txt
cat: can't open '/ありがとう.txt': No such file or directory
```

Becuase the file name is in NFD form, it can only be accessed by `ありか<U+3099>とう.txt`.

## APFS has no such problem

In APFS, [file names are not decomposed to NFD form](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/FAQ/FAQ.html#//apple_ref/doc/uid/TP40016999-CH6-DontLinkElementID_3), 
thus there is no such problem.

To try it, first upgrade to macOS 10.12.4+.

Create a disk:

    hdiutil create -fs APFS -size 1GB foo.sparseimage

Mount it, and `cd` to it in the terminal.

Now try again:

```
$ docker build -t test-image .
$ docker run test-image ありがとう.txt
ありがとう means Thank You!
```

It works :sparkles:

## License

Public Domain
