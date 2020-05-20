# `dumbmv` and `dumbhash`

I had two copies of a collection of large files
(a library of TV episodes).
The files in one copy had been carefully renamed;
the other collection had inconsistent filenames.
I wanted to make the filenames match in both places.
`dumbhash` and `dumbmv` are my solution to this problem.
`dumbhash` generates an `md5sum(1)`-like listing
of the MD5 hashes of the first 512KB of a collection of files.
`dumbmv` compares the hashes between two such listings
(a source and a target),
and renames the files from the source listing
to match the corresponding names from the target.

For example:

    machine-a$ ./dumbhash archive/* | tee targets.txt
    d8c0e766502bb5e19faaebd035d478ac  archive/file-a
    eaef8aa1ecec72410dbe14141e2c6c88  archive/file-b
    machine-a$ scp targets.txt machine-b:

    machine-b$ ./dumbhash archive/* | tee sources.txt
    d8c0e766502bb5e19faaebd035d478ac  archive/a.1080p.WEB-DL.DD5.1.h264-See
    eaef8aa1ecec72410dbe14141e2c6c88  archive/b.1080p.WEB-DL.DD5.1.h264-See
    machine-b$ ./dumbmv sources.txt targets.txt
    Renaming "archive/a.1080p.WEB-DL.DD5.1.h264-See" to "archive/file-a".
    Renaming "archive/b.1080p.WEB-DL.DD5.1.h264-See" to "archive/file-b".

Files missing from either side are ignored.

## Known Issues

Having identified the need for a rename operation,
`dumbmv` directly calls [the Python `os.rename()` library function][os.rename].
This has two caveats:

* The parent directory of the destination filename
  must already exist.
  It will not be automatically created.
  This was fine for me,
  because I was moving files around
  within existing directories.
* `os.rename()` only works when the source and destination
  are on the same filesystem.
  I specifically wanted to avoid accidental copying,
  but you may need to use `shutil.move()` instead.

[os.move]: https://docs.python.org/3/library/os.html#os.rename
[shutil.move]: https://docs.python.org/3/library/shutil.html#shutil.move

## FAQ

**Why not just use the output of `md5sum(1)` directly?**

Because these are big files,
and my hard drives are slow,
and I'm impatient.

**I have small files,
and my hard drives are fast,
and I'm patient.
Can I use the output of `md5sum(1)` directly?**

Yes, that should work correctly.

**MD5 hashes are not cryptographically secure!**

They are fast, though,
and when I'm only considering the first 512KB of the file anyway,
I'm clearly not worried about collisions.
