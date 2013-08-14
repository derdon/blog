Week 9 — Improved Database class
================================
:date: 2013-08-11
:category: gsoc
:summary: The ``Database`` class has been extended by the methods
          ``add_from_path`` and ``add_from_vso_query_result`` and
          ``set_cache_size``. All these methods can be undone using the
          ``undo`` method. The ``query`` method now also supports the
          attributes ``Path``, ``DownloadTime`` and ``FitsHeaderEntry``.

Adding entries by path
----------------------
It is now simpler to add new database entries by passing a directory. The
class ``Database`` got a new method ``add_from_path`` to add all entries
from a given directory. If the optional keyword argument `recursive` is
true, the given directory is searched recursively.

.. code-block:: pycon

    >>> from sunpy.database import Database
    >>> database = Database('sqlite:///:memory:')
    >>> database.create_tables()
    >>> import sunpy.data.sample
    >>> database.add_from_path(sunpy.data.sample.rootdir)
    >>> len(database)
    5
    >>> from sunpy.database import display_entries
    >>> print display_entries(database, ['id', 'observation_time_start', 'observation_time_end', 'instrument', 'waveunit', 'wavemin', 'wavemax'])
    id observation_time_start     observation_time_end       instrument waveunit wavemin wavemax
    -- ----------------------     --------------------       ---------- -------- ------- -------
    1  2010-10-16 19:12:18        2010-10-16 19:12:22        RHESSI     N/A      N/A     N/A    
    2  2002-06-25 10:00:10.514000 N/A                        EIT        N/A      195.0   195.0  
    3  2011-03-19 10:54:00.340000 N/A                        AIA_3      N/A      171.0   171.0  
    4  2011-09-22 00:00:00        2011-09-22 00:00:00        BIR        N/A      N/A     N/A    
    5  2002-02-20 11:06:00        2002-02-20 11:06:43.330000 RHESSI     N/A      N/A     N/A    

Adding entries by VSO query result
----------------------------------
To add entries from a VSO query result, the API has been simplified: the
new method ``add_from_vso_query_result`` accepts a query result and all
reachable information will be used to add new entries to the database.

.. code-block:: pycon

    >>> from sunpy.net import vso
    >>> client = vso.VSOClient()
    >>> qr = client.query(vso.attrs.Time('2010-10-01 10:10:01', '2010-10-01 10:10:10'), vso.attrs.Instrument('AIA'))
    >>> qr.num_records()
    6
    >>> database.add_from_vso_query_result(qr)
    >>> len(database)
    11
    >>> print display_entries(database, ['id', 'observation_time_start', 'observation_time_end', 'instrument', 'waveunit', 'wavemin', 'wavemax'])
    id observation_time_start     observation_time_end       instrument waveunit wavemin wavemax
    -- ----------------------     --------------------       ---------- -------- ------- -------
    1  2010-10-16 19:12:18        2010-10-16 19:12:22        RHESSI     N/A      N/A     N/A    
    2  2002-06-25 10:00:10.514000 N/A                        EIT        N/A      195.0   195.0  
    3  2011-03-19 10:54:00.340000 N/A                        AIA_3      N/A      171.0   171.0  
    4  2011-09-22 00:00:00        2011-09-22 00:00:00        BIR        N/A      N/A     N/A    
    5  2002-02-20 11:06:00        2002-02-20 11:06:43.330000 RHESSI     N/A      N/A     N/A    
    6  2010-10-01 10:10:02        2010-10-01 10:10:03        AIA        Angstrom 94.0    94.0   
    7  2010-10-01 10:10:03        2010-10-01 10:10:04        AIA        Angstrom 335.0   335.0  
    8  2010-10-01 10:10:06        2010-10-01 10:10:07        AIA        Angstrom 193.0   193.0  
    9  2010-10-01 10:10:07        2010-10-01 10:10:08        AIA        Angstrom 1700.0  1700.0 
    10 2010-10-01 10:10:08        2010-10-01 10:10:09        AIA        Angstrom 304.0   304.0  
    11 2010-10-01 10:10:09        2010-10-01 10:10:10        AIA        Angstrom 131.0   131.0  

Changing cache size at runtime
------------------------------
The cache size can now be changed at runtime. If the cache size becomes
smaller, entries are removed until ``database.cache_size`` equals
``database.cache_maxsize``. Which entries are removed depends on the
actual cache implementation.

.. code-block:: pycon

    >>> # "touch" some entries to make sure that they're not removed
    >>> _, _, _,  _, = database[2], database[5], database[7], database[9]
    >>> database.set_cache_size(5)
    >>> print display_entries(database, ['id', 'observation_time_start', 'observation_time_end', 'instrument', 'waveunit', 'wavemin', 'wavemax'])
    id observation_time_start     observation_time_end instrument waveunit wavemin wavemax
    -- ----------------------     -------------------- ---------- -------- ------- -------
    3  2011-03-19 10:54:00.340000 N/A                  AIA_3      N/A      171.0   171.0  
    6  2010-10-01 10:10:02        2010-10-01 10:10:03  AIA        Angstrom 94.0    94.0   
    8  2010-10-01 10:10:06        2010-10-01 10:10:07  AIA        Angstrom 193.0   193.0  
    10 2010-10-01 10:10:08        2010-10-01 10:10:09  AIA        Angstrom 304.0   304.0  
    11 2010-10-01 10:10:09        2010-10-01 10:10:10  AIA        Angstrom 131.0   131.0  

New query attributes: Path, DownloadTime, FitsHeaderEntry
---------------------------------------------------------
The ``query`` method now also supports the attributes ``Path``,
``DownloadTime``, and ``FitsHeaderEntry``.

.. code-block:: pycon

    >>> entries = database.query(db_attrs.FitsHeaderEntry('WAVELNTH', 171))
    >>> len(entries)
    1
    >>> print display_entries(entries, ['id', 'waveunit', 'wavemin', 'wavemax'])
    id waveunit wavemin wavemax
    -- -------- ------- -------
    3  N/A      171.0   171.0  
