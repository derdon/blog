Week 5 — Importing database entries from a directory
====================================================
:date: 2013-07-08
:category: gsoc
:summary: A new feature is to make new database entries from a given
          directory.

The class ``DatabaseEntry`` now has a new classmethod:
``from_fits_filepath``. It receives a path to a FITS filename and parses
its header to make a new instance of ``DatabaseEntry``. All key-value
pairs can be found in the attribute ``fits_header_entries``, which is a
list of ``FitsHeaderEntry`` instances. Additionally, this classmethod
searches for the keys *INSTRUME*, *WAVELNTH*, *DATE-OBS* / *DATE_OBS*, and
*DATE-END* / *DATE_END*. Their values are used to set the attribute
``instrument``, ``wavemin`` and ``wavemax``, ``observation_time_start``
or ``observation_time_end``, respectively.

.. code-block:: pycon

    >>> from sunpy.database import DatabaseEntry
    >>> entry = DatabaseEntry.from_fits_filepath('sunpy/data/efz20010101.143610.fits')
    >>> entry.instrument
    'EIT'
    >>> entry.wavemin, entry.wavemax
    (195.0, 195.0)
    >>> entry.observation_time_start, entry.observation_time_end
    (datetime.datetime(2001, 1, 1, 14, 36, 10, 983000), None)
    >>> for header_entry in entry.fits_header_entries:
    ...     if header_entry.key in ('INSTRUME', 'WAVELNTH', 'DATE-OBS'):
    ...         print header_entry
    ... 
    <FitsHeaderEntry(id None, key 'DATE-OBS', value '2001-01-01T14:36:10.983')>
    <FitsHeaderEntry(id None, key 'INSTRUME', value 'EIT')>
    <FitsHeaderEntry(id None, key 'WAVELNTH', value 195)>

There is also a new utility function called ``entries_from_path`` to get
an iterator of ``DatabaseEntry`` instances by passing a directory path.
This function also accepts the optional arguments `recursive` and
`pattern`. The former argument is boolean and determines whether to search
the directory recursively or not. The latter one is a string and defines
how a FITS path is detected by its filename. This value is passed to
`fnmatch.filter <http://docs.python.org/2.7/library/fnmatch.html#fnmatch.filter>`_
and its default is *\*.fits*. Again, the values of the attributes
``instrument``, ``wavemin``, ``wavemax``, ``observation_time_start``, and
``observation_time_end`` are set, if possible.

.. code-block:: pycon

    >>> from sunpy.database import entries_from_path
    >>> from sunpy.data.test import rootdir as sampledir
    >>> entries = list(entries_from_path(os.path.join(sampledir, 'EIT')))
    >>> len(entries)
    13
    >>> first_entry, path = entries[0]
    >>> first_entry.instrument
    'EIT'
    >>> first_entry.wavemin, first_entry.wavemax
    (195.0, 195.0)
    >>> first_entry.observation_time_start, first_entry.observation_time_end
    (datetime.datetime(2004, 3, 1, 2, 0, 10, 642000), None)
