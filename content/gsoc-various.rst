Week 8 — Various new features
=============================
:date: 2013-08-04
:category: gsoc
:summary: In this week I added different new features for which I can't
          find one headline. The slicing syntax for ``Database`` and the
          function ``display_entries`` have been implemented thanks to
          one of my mentors, Steven Christe. Thank you for the ideas!

Preparation
-----------
To be able to show the new implemented features, the database needs to be
initialized and populated with some entries. The VSO is queried for all
data that has been measured on 2013-08-04 between 12:00:00 and 12:10:00.
You can see that this queried data has been completely added to the
database, because ``qr.num_records()`` returns the same number as
``len(database)``.

.. code-block:: pycon

    >>> from sunpy.database import Database, entries_from_query_result
    >>> from sunpy.net import vso
    >>> client = vso.VSOClient()
    >>> qr = client.query(vso.attrs.Time('20130804T120000', '20130804T121000'))
    >>> qr.num_records()
    61
    >>> database = Database('sqlite:///:memory:')
    >>> database.create_tables()
    >>> for entry in entries_from_query_result(qr):
    ...     database.add(entry)
    ...     
    ... 
    >>> len(database)
    61

Slicing syntax for ``Database`` instances
-----------------------------------------
I added a handy and readable syntax for fetching some database entries
without using the query interface: the slice syntax. Entries are returned
ordered by their unique ID and in my opinion it works as expected (but I
need feedback from more SunPy devs and users to be sure on that). Take a
look at the following code snippet to see how it works:

.. code-block:: pycon

    >>> from pprint import pprint
    >>> assert database[:] == list(database)
    >>> pprint(database[:6])
    [<DatabaseEntry(id 1, data provider NSO, fileid pptid=11007;url=ftp://gong2.nso.edu/oQR/bqa/201308/tdbqa130804/tdbqa130804t1204.fits.gz)>,
     <DatabaseEntry(id 2, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120014Th.fits.fz)>,
     <DatabaseEntry(id 3, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120114Th.fits.fz)>,
     <DatabaseEntry(id 4, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120214Th.fits.fz)>,
     <DatabaseEntry(id 5, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120314Th.fits.fz)>,
     <DatabaseEntry(id 6, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120414Th.fits.fz)>]
    >>> pprint(database[:6:2])
    [<DatabaseEntry(id 1, data provider NSO, fileid pptid=11007;url=ftp://gong2.nso.edu/oQR/bqa/201308/tdbqa130804/tdbqa130804t1204.fits.gz)>,
     <DatabaseEntry(id 3, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120114Th.fits.fz)>,
     <DatabaseEntry(id 5, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120314Th.fits.fz)>]
    >>> pprint(database[5:0:-2])
    [<DatabaseEntry(id 6, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120414Th.fits.fz)>,
     <DatabaseEntry(id 4, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120214Th.fits.fz)>,
     <DatabaseEntry(id 2, data provider NSO, fileid pptid=11018;url=ftp://gong2.nso.edu/HA/haf/201308/20130804/20130804120014Th.fits.fz)>]

Display entries in a table
--------------------------
The new function ``display_entries`` accepts an iterable of
``DatabaseEntry`` instances and an iterable of strings which stand for the
column names of the main database table. Keep in mind that
``DatabaseEntry`` is a mapping from a class to a database table, which
means that the passed strings are translated to the respective attributes
of the entry to get the corresponding value in the database. The code in
the following example prints the ID, the instrument, the wavelength unit,
the minimum wavelength and the maximum wavelength of every tenth database
entry, beginning with the entry #10:

.. code-block:: pycon

    >>> print display_entries(database[9::10], ['id', 'instrument', 'waveunit', 'wavemin', 'wavemax'])
    id instrument waveunit wavemin wavemax
    -- ---------- -------- ------- -------
    10 El teide   Angstrom 6562.0  6563.0 
    20 ChroTel    Angstrom 6562.0  6562.0 
    30 AIA        Angstrom 304.0   304.0  
    40 AIA        Angstrom 304.0   304.0  
    50 AIA        Angstrom 335.0   335.0  
    60 AIA        Angstrom 1600.0  1600.0 

Removing tags
-------------
To remove a tag of a certain entry, the method ``remove_tag`` is used
which accepts the entry that should “lose” a tag and the name of the tag
that is to be removed. The propertry ``tags`` of a ``Database`` instance
returns a list of all tags that are saved in the database. It is important
to know that if a tag is removed from an entry and there is no database
entry in the database with such a tag assigned, this tag itself is removed
from the database as well! You can check this behaviour in the following
example:

.. code-block:: pycon

    >>> database.tags
    []
    >>> first_entry, second_entry = database[:2]
    >>> database.tag(first_entry, 'one')
    >>> database.tag(second_entry, 'one', 'two')
    >>> database.tags
    [<Tag(id 1, name 'one')>, <Tag(id 2, name 'two')>]
    >>> database.remove_tag(second_entry, 'two')
    >>> database.tags
    [<Tag(id 1, name 'one')>]
    >>> database.remove_tag(first_entry, 'one')
    >>> database.tags
    [<Tag(id 1, name 'one')>]
    >>> database.remove_tag(second_entry, 'one')
    >>> database.tags
    []

Supported VSO attributes for querying the database
--------------------------------------------------
Currently, only one kind of VSO attribute is supported for querying the
database: simple attributes. Simple attributes are those which have only
one single value assigned. In particular, they are:

    - Instrument

    - Source

    - Provider

    - Physobs

The following example queries the database for all entries with the
instrument `ChroTel`, the provider `KIS` and the physobs `Intensity`:

.. code-block:: pycon

    >>> print display_entries(
    ...     database.query(vso.attrs.Instrument('ChroTel'), vso.attrs.Provider('KIS'), vso.attrs.Physobs('Intensity')),
    ...     ['id', 'waveunit', 'wavemin', 'wavemax'])
    id waveunit wavemin wavemax
    -- -------- ------- -------
    13 Angstrom 3934.0  3934.0 
    14 Angstrom 6562.0  6562.0 
    15 Angstrom 10830.0 10830.0
    16 Angstrom 3934.0  3934.0 
    17 Angstrom 6562.0  6562.0 
    18 Angstrom 10830.0 10830.0
    19 Angstrom 3934.0  3934.0 
    20 Angstrom 6562.0  6562.0 
    21 Angstrom 10830.0 10830.0
    22 Angstrom 3934.0  3934.0 
    23 Angstrom 6562.0  6562.0 
    24 Angstrom 10830.0 10830.0
