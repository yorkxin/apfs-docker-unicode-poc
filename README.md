# APFS Japanese Kana Compatibility PoC

## Problem of Kana File Name

In HFS+, file name will be decomposed to Unicode NFD form. So a file named `ハンバーガー` will be decomposed to `ハンハ<U+3099>ーカ<U+3099>ー`,
where [U+3099 is a 濁点 sound mark](https://codepoints.net/U+3099).

When we build an image that is based on Linux, Docker doesn't compose the string back to NFC form, 
thus any file with 濁点 or 半濁点 will not be accessible.

For example:

```
$ docker build -t test-image .
$ docker run test-image /test.sh
cat: can't open '/ハンバーガー.txt': No such file or directory
```

Becuase the file name is in NFD form, it can only be accessed by `ハンハ<U+3099>ーカ<U+3099>ー.txt`.

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
$ docker run test-image /test.sh
🍔
```

It works :sparkles:

## For files checked in from HFS+

Actually Git will convert the file name to NFC, so it still works in Linux:

```
$ docker run test-image /test.sh
🍔
ひらがな added from HFS
```

## See also

* [Frequently Asked Questions](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/FAQ/FAQ.html#//apple_ref/doc/uid/TP40016999-CH6-DontLinkElementID_3)
* [日本の文字とUnicode　第3回 | 大修館書店　WEB国語教室](http://www.taishukan.co.jp/kokugo/webkoku/series003_03.html)

## License

Public Domain
