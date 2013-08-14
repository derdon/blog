Week 6 — Querying the database
==============================
:date: 2013-07-18
:category: gsoc
:summary: It is now possible to query the database.

Intro: Comparison with the VSO query interface
----------------------------------------------
Querying the database works like the query method of VSOClient. The
following query call filters all the data that was observed on 1st
January, 2010 from 0:00 to 1:00 and was observed via the instrument EIT or
SOT:

.. code-block:: python

    result = client.query(
       vso.attrs.Time((2010, 1, 1), (2010, 1, 1, 1)),
       vso.attrs.Instrument('eit') | vso.attrs.Instrument('sot')
    )

Preparing the database: adding sample entries
---------------------------------------------
Before showing how to query a database, the database is prepared by adding
some sample entries. Ten entries are added, of which every second one is
starred, every fourth one gets the tag *foo* and every fifth one gets the
tag *bar*. To make sure that these entries are saved in the database, the
method ``commit`` is called on the database object.

.. code-block:: pycon

    >>> from sunpy.database import Database, DatabaseEntry
    >>> database = Database('sqlite:///:memory:')
    >>> database.create_tables()
    >>> for i in xrange(1, 11):
    ...     entry = DatabaseEntry()
    ...     database.add(entry)
    ...     if i % 2 == 0:
    ...         database.star(entry)
    ...     if i % 4 == 0:
    ...         database.tag(entry, 'foo')
    ...     if i % 5 == 0:
    ...         database.tag(entry, 'bar')
    ... 
    >>> database.commit()

The actual part: querying the database
--------------------------------------
To query a database (an instance of the class
``sunpy.database.Database``), its method ``query`` is used. It accepts any
number (though at least one) of attributes which are chained together
using the logical AND-operator. To use OR, the operator ``|`` may be used.
The first example in the following snippet gets all starred entries with
the tag *foo*. The second one gets all entries with the tag *bar* that are
**not** starred. Note the tilde ``~`` before the attribute to negate the
meaning of this attribute. The third and last query in this example
fetches all starred entries with the tag *foo* or *bar* (or both!) and
sorts the result by their unique ID. The optional keyword argument `id` is
a string that expresses by which column to sort.

.. code-block:: pycon

    >>> from sunpy.database import attrs
    >>> database.query(attrs.Tag('foo'), attrs.Starred())
    [<DatabaseEntry(id 4, data provider None, fileid None)>, <DatabaseEntry(id 8, data provider None, fileid None)>]
    >>> database.query(attrs.Tag('bar'), ~attrs.Starred())
    [<DatabaseEntry(id 5, data provider None, fileid None)>]
    >>> from pprint import pprint
    >>> pprint(database.query(attrs.Tag('foo') | attrs.Tag('bar'), attrs.Starred(), sortby='id'))
    [<DatabaseEntry(id 4, data provider None, fileid None)>,
     <DatabaseEntry(id 8, data provider None, fileid None)>,
     <DatabaseEntry(id 10, data provider None, fileid None)>]

What are the next plans?
------------------------
The next plan is to support query attributes from the VSO package so that
code like the following will be possible:

.. code-block:: pycon

    >>> from sunpy.net import vso
    >>> database.query(vso.attrs.Instrument('EIT'), vso.attrs.Time('2000-01-01', '2000-01-31'))

This would fetch all data that was observed with the EIT in the January
2000.
