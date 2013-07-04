Week 2 — Database Entries
=========================
:date: 2013-06-22
:category: gsoc
:summary: The class ``DatabaseEntry`` is the main table in the database.
          Each instance of it is an entry in the database and contains all
          relevant information in its attributes.

The DatabaseEntry class
-----------------------
In the last blog post, I have used the class ``DatabaseEntry`` without
really explaining it. That may have confused the reader but I had a reason
for doing so: the public API was not final at all. The class didn't have
any methods and the attributes were not decided yet. The class
``DatabaseEntry`` represents the main table of the database and each
instance of this class represents one database entry. Each attribute is
one field in the database table.

adding information from a FITS file
-----------------------------------
I added a new method ``add_fits_header_entries_from_file``. With it, the
information from a FITS file can easily be added to a database entry. FITS
files usually contain a lot of metadata about the observation like the
time it took place, which instrument has been used etc. The passed
argument may be a path to a FITS file or any file-like object (e.g. an
instance of StringIO.StringIO). The saved information is saved in the
attribute ``fits_header_entries`` which is a list of ``FitsHeaderEntry``
instances. The class ``FitsHeaderEntry`` holds the attributes ``id``,
``key``, and ``value``. Here is an example:

.. code-block:: pycon

    >>> from pprint import pprint
    >>> import sunpy
    >>> from sunpy.database import DatabaseEntry
    >>> entry = DatabaseEntry()
    >>> entry.add_fits_header_entries_from_file(sunpy.RHESSI_EVENT_LIST)
    >>> pprint(entry.fits_header_entries)
    [<FitsHeaderEntry(id None, key 'SIMPLE', value True)>,
     <FitsHeaderEntry(id None, key 'BITPIX', value 8)>,
     <FitsHeaderEntry(id None, key 'NAXIS', value 0)>,
     <FitsHeaderEntry(id None, key 'EXTEND', value True)>,
     <FitsHeaderEntry(id None, key 'DATE', value '2011-09-13T15:37:38')>,
     <FitsHeaderEntry(id None, key 'ORIGIN', value 'RHESSI')>,
     <FitsHeaderEntry(id None, key 'OBSERVER', value 'Unknown')>,
     <FitsHeaderEntry(id None, key 'TELESCOP', value 'RHESSI')>,
     <FitsHeaderEntry(id None, key 'INSTRUME', value 'RHESSI')>,
     <FitsHeaderEntry(id None, key 'OBJECT', value 'Sun')>,
     <FitsHeaderEntry(id None, key 'DATE_OBS', value '2002-02-20T11:06:00.000')>,
     <FitsHeaderEntry(id None, key 'DATE_END', value '2002-02-20T11:06:43.330')>,
     <FitsHeaderEntry(id None, key 'TIME_UNI', value 1)>,
     <FitsHeaderEntry(id None, key 'ENERGY_L', value 25.0)>,
     <FitsHeaderEntry(id None, key 'ENERGY_H', value 40.0)>,
     <FitsHeaderEntry(id None, key 'TIMESYS', value '1979-01-01T00:00:00')>,
     <FitsHeaderEntry(id None, key 'TIMEUNIT', value 'd')>]

generating entries by a VSO query response
------------------------------------------
With the ``sunpy.net.vso`` package, solar data can be queried and
downloaded. If a query is issued, a query response is returned. This query
response can be used to create an iterator of database entries. They are
not yet saved in a database though. To save them, a database connection
has to be established and the entries have to be added explicitly. See the
last post about the ``Database`` class for more information. Also see `the
documentation of the VSO package
<http://sunpy.readthedocs.org/en/latest/guide/vso.html>`_ for more
information about its usage.

.. code-block:: pycon

    >>> from sunpy.net.vso import VSOClient
    >>> from sunpy.database import entries_from_query_result
    >>> client = VSOClient()
    >>> qr = client.query_legacy('2001/1/1', '2001/1/2', instrument='EIT')
    >>> entries = entries_from_query_result(qr)
    >>> entries.next()
    <DatabaseEntry(id None, data provider SDAC, fileid /archive/soho/private/data/processed/eit/lz/2001/01/efz20010101.010014)>
