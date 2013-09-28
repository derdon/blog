End of Google Summer of Code
============================
:date: 2013-09-28
:category: gsoc
:summary: GSoC is over. I'm glad to have participated. I will continue
          contributing to the SunPy project and even consider mentoring
          next year.

Thank You!
----------
Yesterday was the last day of my Google Summer of Code project. I want to
use this blog post to say thank you. Thank you Google for making this
program possible and thank you Carol Smith for the great help and
administrative work throughout the whole GSoC 2013. Thanks to my mentor,
Florian Mayer, for helping me understand the existing VSO package code and
for general guidance and tips. Thanks to Steven Christe for various ideas,
especially the function ``display_entries`` and the ability to fetch a
subset of all entries by using the slicing syntax. 

Plans
-----
The database package is not finished to a 100%; it has at least one bug
that I know of (but I'm pretty there are more that will be found only if
this package is used by real users), documentation can be improved and
some features need to be added.

The bug that I know of is that the ``undo`` method of the ``Database``
class is broken for the method ``set_cache_size`` if it reduces the number
of database entries. But I already know where the problem is, so it should
be easy to fix: removing the database entries can be undone, but the
original internal cache is not saved so it is not restored after the
``undo`` call.

One thing that I couldn't commit to my database branch was the feature of
dumping and loading of database entries to a single text file. Use cases
for this feature is the possibility to share this file with other people
and also use it as a backup.

Another thing I want to implement is the support of adding database
entries by an HEK query result. Thanks to the work of Michael Malocha (the
other GSoC student working for SunPy) this should be quite simple because
he wrote a module which can translate VSO queries to HEK queries.

Currently, only FITS files are supported in the ``download`` method of the
``Database`` class. But the files that will be downloaded via
``VSOClient.get`` may also be \*.tar or \*.tar.gz files, so I plan to
support these kind of files as well.
