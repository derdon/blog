Week 3 — Tag Support
====================
:date: 2013-06-30
:category: gsoc
:summary: It is now possible to assign tags to entries.

adding tags
-----------
Tags are usually single words that describe entries in a brief way.
Although it is possible to use tags that include whitespace or are
very long, it is not recommended to do so. To add new tags to an entry,
use the method ``tag`` from the ``Database`` class. It receives an
instance of ``DatabaseEntry`` (the entry that shall get assigned new tags)
and at least one tag name. The new tags are stored in the attribute
``tags`` of this modified database entry. This is a list of ``Tag``
instances which contain a unique ID and the name of the tag (which is
also unique). The ``Database`` class has also an attribute called
``tags``. This stores *all* tags that have been saved in the database.

In the following example, you can see that the attributes ``tags`` of both
the database object and of the entry are by default empty lists. After the
created entry is tagged with the tags “shiny”, “bright”, and “yellow”, the
``tags`` attribute of entry is not empty anymore but contains a list of
``Tag`` instances, as expected. But why is the list ``database.tags``
still empty? The answer: because the entry is not saved in the database
yet. It must first be added with the ``add`` method. After that, the tags
are saved in the database as we can see in the ID information (if a
database-related object such as instances of ``DatabaseEntry`` or ``Tag``
does not have an ID, it is not stored in the database).

.. code-block:: pycon

    >>> from pprint import pprint
    >>> from sunpy.database import Database, DatabaseEntry
    >>> database = Database('sqlite:///:memory:')
    >>> database.create_tables()
    >>> entry = DatabaseEntry()
    >>> database.tags
    []
    >>> entry.tags
    []
    >>> database.tag(entry, 'shiny', 'bright', 'yellow')
    >>> entry.tags
    [<Tag(id None, name 'bright')>, <Tag(id None, name 'shiny')>, <Tag(id None, name 'yellow')>]
    >>> database.tags
    []
    >>> database.add(entry)
    >>> database.tags
    [<Tag(id 1, name 'bright')>, <Tag(id 2, name 'shiny')>, <Tag(id 3, name 'yellow')>]

get entries by tag(s)
---------------------
To get all entries that have certains tags assigned, the ``Database``
class has the new method ``get_by_tags``. It receives at least one tag
name (that means a string, not an instance of ``Tag``!) and returns a list
of ``DatabaseEntry`` instances which have *at least* one of the given tags
assigned.

What about removing tags?
-------------------------
Removing tags is not implemented yet. Because there is a many-to-many
relationship, this is not trivial. I will have to read `Deleting Rows from
the Many to Many Table 
<http://docs.sqlalchemy.org/en/rel_0_8/orm/relationships.html#deleting-rows-from-the-many-to-many-table>`_
from the SQLAlchemy Documentation to find out what to consider when
removing entries in a many-to-many relationship.
