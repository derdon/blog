Week 11 — Support for the VSO attributes Wave and Time
======================================================
:date: 2013-08-23
:category: gsoc
:summary: The attribute ``Wave`` now searches all entries and converts
          wavelengths from the given unit to nm, if necessary. A new
          attribute that is now supported is ``Time``.

Preparation: filling the database with entries
----------------------------------------------
The VSO is queried for all AIA data that has been observed on the 8th of
June, 2011 from 23:59:55 to the end of this day (i.e. a range of 5
seconds). The result is added to the database and displayed in a table to
show the values of the observation time (start and end) and the
wavelengths. You can see that the wavelengths have been converted from
Angstrom to Nanometers. The reason for converting all wavelengths to
Nanometers before saving them is that it makes querying using the ``Wave``
attribute quite convenient (see the next section to better understand this
claim).

.. code-block:: pycon

    >>> from sunpy.net import vso
    >>> client = vso.VSOClient()
    >>> qr = client.query(vso.attrs.Time('20110608T235955', '2011-06-09'), vso.attrs.Instrument('aia'))
    >>> for block in qr:
    ...     print '{wave.wavemin}\t{wave.wavemax}\t{wave.waveunit}\t'.format(wave=block.wave)
    ... 
    193     193     Angstrom
    94      94      Angstrom
    131     131     Angstrom
    171     171     Angstrom
    211     211     Angstrom
    >>> from sunpy.database import Database
    >>> database = Database('sqlite:///:memory:')
    >>> database.create_tables()
    >>> database.add_from_vso_query_result(qr)
    >>> from sunpy.database.tables import display_entries
    >>> print display_entries(
    ...     database,
    ...     ['id', 'observation_time_start', 'observation_time_end', 'wavemin', 'wavemax'])
    id observation_time_start observation_time_end wavemin wavemax
    -- ---------------------- -------------------- ------- -------
    1  2011-06-08 23:59:55    2011-06-08 23:59:56  19.3    19.3   
    2  2011-06-08 23:59:56    2011-06-08 23:59:57  9.4     9.4    
    3  2011-06-08 23:59:57    2011-06-08 23:59:58  13.1    13.1   
    4  2011-06-09 00:00:00    2011-06-09 00:00:01  17.1    17.1   
    5  2011-06-09 00:00:00    2011-06-09 00:00:01  21.1    21.1   

Querying for wavelengths
------------------------
Previously, querying for certain wavelengths such as ``Wave(50, 80, 'GHz')``
would search for all entries *with the waveunit GHz (Gigahertz)* and then
check if the wavelengths of these entries are in the interval [50, 80].
Now, such a query will iterate over *all* entries and convert the values
50GHz and 80GHz to Nanometers and check for each database entry if its
wavelengths are in the converted range. Here is an example using
Angstroms:

.. code-block:: pycon

    >>> print display_entries(
    ...     database.query(vso.attrs.Wave(100, 200)),
    ...     ['id', 'observation_time_start', 'observation_time_end', 'wavemin', 'wavemax'])
    id observation_time_start observation_time_end wavemin wavemax
    -- ---------------------- -------------------- ------- -------
    1  2011-06-08 23:59:55    2011-06-08 23:59:56  19.3    19.3   
    3  2011-06-08 23:59:57    2011-06-08 23:59:58  13.1    13.1   
    4  2011-06-09 00:00:00    2011-06-09 00:00:01  17.1    17.1      

Querying for observation times
------------------------------
A new supported attribute for querying the database is ``vso.attrs.Time``.
The start of the observation time of each yielded entry is greater than or
equal to the `start` parameter of the ``Time`` attribute. The end of the
observation time of each yielded entry is lower than or equal to the `end`
parameter of the ``Time`` attribute. See the following example to see how
it works:

.. code-block:: pycon

    >>> print display_entries(
    ...     database.query(vso.attrs.Time('2011-06-08 23:59:57', '2011-06-10')),
    ...     ['id', 'observation_time_start', 'observation_time_end', 'wavemin', 'wavemax'])
    id observation_time_start observation_time_end wavemin wavemax
    -- ---------------------- -------------------- ------- -------
    3  2011-06-08 23:59:57    2011-06-08 23:59:58  13.1    13.1   
    4  2011-06-09 00:00:00    2011-06-09 00:00:01  17.1    17.1   
    5  2011-06-09 00:00:00    2011-06-09 00:00:01  21.1    21.1
