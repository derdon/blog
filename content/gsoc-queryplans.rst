Week 4 — Plans for query support
================================
:date: 2013-07-04
:category: gsoc
:summary: One big feature that is still missing is querying the database,
          i.e. reading from the databaase by sending queries. In this
          post, I will share my ideas and plans to implement this feature.

User perspective first!
-----------------------
How does querying work with the existing VSO interface?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Querying a VSO server with the VSO interface of SunPy can look like this:

.. code-block:: pycon

    >>> from sunpy.net import vso
    >>> client = vso.VSOClient()
    >>> qr = client.query(vso.attrs.Time('2001/1/1', '2001/1/2'), vso.attrs.Instrument('eit'))

The ``query`` method of ``VSOClient`` receives any number of attributes
and conjugates them. So if you read the exemplary query above, it reads
like “query all the data that was observed from 2001/1/1 to 2001/1/2 *and*
was observed using the instrument EIT”. ``query`` returns an instance of
``sunpy.net.vso.vso.QueryResponse`` which can be used to download the
queried data.

How should querying the database look like?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To keep the usage consistent with the query interface of the VSO package
described above, I plan to make the database query interface usable like
this:

.. code-block:: pycon

    >>> from sunpy.net.vso import attrs as vso_attrs
    >>> from sunpy.database import Database
    >>> from sunpy.database import attrs as db_attrs
    >>> database = Database('sqlite:///:memory:')
    >>> entries = database.query(vso_attrs.Instrument('eit'), db_attrs.Tag('sun'), db_attrs.Starred(True))

The class ``Database`` gets a new method called ``query`` which accepts
any number of attributes and returns a generator of ``DatabaseEntry``
instances. I hope you can see the similarities to the VSO query interface.
Apart from the VSO attributes from the module ``sunpy.net.vso.attrs``,
this query method will also support database-specific attributes such as
``Tag`` or ``Starred``. Equivalent to the VSO query method, this one also
conjugates the passed attributes before they are processed, so the query
above can be read as “give me all entries that were observed with an EIT
instrument *and* are tagged 'sun' *and* are starred”.

**Edit**: After a night of good sleep, I think I have come to a better API
for the ``Starred`` attribute. Or, more general, for any boolean
attribute:

.. code-block:: pycon

    >>> starred_entries = database.query(db_attrs.Starred())
    >>> unstarred_entries = database.query(~db_attrs.Starred())

The iterator ``starred_entries`` will contain all entries that have been
marked as starred, whereas ``unstarred_entries`` will contain all
non-starred entries.

The implementation
------------------
To make the described plans possible, one new module within the database
package will be needed: it will be called ``attrs`` (to keep it consistent
with the VSO package) and will define how to convert attributes to database
queries. Additionally, it will introduce the new attributes ``Tag`` and
``Starred``.

.. The module ``sunpy.net.attr`` defines a class ``AttrWalker`` ...

.. FIXME
    new module: sunpy.database.attrs
    --------------------------------
    - use attributes from sunpy.net.vso.attrs
    - create an attribute walker by importing sunpy.net.attr.AttrWalker and
      creating an instance of it
    - use this walker to define creators, appliers and converters using the
      decorators add_creator, add_applier, and add_converter, respectively
    class Database
    --------------
    - add a new method ``query`` which receives any number of attributes and
      yields instances of DatabaseEntry → implementation: iterate over
      ``walker.create(sunpy.net.attr.and_(*query), self.session)``
