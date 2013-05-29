GSOC 2013 -- Initial Post
=========================
:date: 2013-05-28
:category: gsoc
:summary: I have found out that I am accepted for GSOC and am pretty
          excited!

So yesterday evening I found out that I am one of the participating
students of GSOC and this is really cool! Wait, what is “GSOC” and why am
I so excited about it? GSOC is short for Google Summer of Code and is
stipend program by Google to improve Open Source projects. The Open Source
project I have applied for is SunPy, a Python package for gathering and
analyzing astronomical data concerning solar physics. In my project, I
will add a database interface to the sunpy package so that information
about downloaded data is stored in a central database (by default on the
local hard drive, possibly also remote servers will be supported). This 
makes it possible to query the database, e.g. to display all data that has
been downloaded in the last two weeks. The database will act as a cache,
meaning that one gets the data from the database if a download from a
source is attempted that was already downloaded before.
