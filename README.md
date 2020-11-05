# `dumbhash`, `dumbmv`, and `dumbln`

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

`dumbln` came about later as the solution
to a similar but slightly different problem:
I had two copies of a collection of large files
(a download directory and a media directory),
and I wanted to [hard-link][hard link] duplicates to each other
to reduce disk space.
`dumbln` compares the hashes
between one or more `dumbhash` listings
and replaces all of the paths for duplicate data
with links to the same inode.
It will keep inode with the lowest number,
and reset its mtime/atime to be the min/max
of the mtimes/atimes of all of the duplicates.

For example:

    $ ./dumbhash downloads/* | tee downloads.txt
    d8c0e766502bb5e19faaebd035d478ac  downloads/a.1080p.WEB-DL.DD5.1.h264-See
    $ ./dumbhash media/* | tee media.txt
    d8c0e766502bb5e19faaebd035d478ac  media/file-a
    $ ./dumbln downloads.txt media.txt
    Linking "downloads/a.1080p.WEB-DL.DD5.1.h264-See" to "media/file-a".

Unique files are ignored,
as are files where all paths already point to the same inode.

[hard link]: https://en.wikipedia.org/wiki/Hard_link

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

If the same hash appears multiple times
in either `dumbmv` input file,
only the occurrence will be considered.
`dumbln` shouldn't have this problem.

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
