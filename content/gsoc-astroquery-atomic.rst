Astroquery -- New Service Atomic Line List
==========================================
:date: 2014-08-04
:category: gsoc
:summary: The new support for the service will allow the user to ...

"Atomic Line List" is a collection of more than 900,000 atomic transitions in
the range from 0.5 Å to 1000 µm (source_). By adding support for this
service in astroquery, it will be possible to access these records easily
with the Python programming language.

The class ``AtomicLineList`` has only 2 public methods: ``query_object``
and ``query_object_async``. The latter one only gives us an HTTP response
object, whereas the former one converts the HTTP response into an AstroPy
table. So let's take a look at ``query_object``: So far, it only has four
parameters (all optional): ``wavelength_range``, ``wavelength_type``,
``wavelength_accuracy`` and ``element_spectrum``. The respective web form
for Atomic Line List can be found at http://www.pa.uky.edu/~peter/atomic/.
As you can see there, the first form fields are "Wavelength range" and
"Unit". The AstroPy_ package offers a very handy `unit` package, so I
decided to support this type instead of passing plain strings. Therefore,
even more units are supported than the one given in the web form! The
parameter `wavelength_range` is a 2-tuple where each item is a scalar
multiplied with an AstroPy unit. Behind the scenes, both values will be
converted to Angstrom (arbitrarily chosen from the web form's dropdown
menu)

In the following Python session you can see the atomic package in action.
Note that Hz is actually not a supported unit by Atomic Line List, the
atomic package takes care to support all spectral units.

.. code-block:: pycon

    >>> from astropy import units as u
    >>> from astroquery.atomic import AtomicLineList
    >>> alist = AtomicLineList()
    >>> wavelength_range = (15 * u.nm, 1.5e+16 * u.Hz)
    >>> alist.query_object(wavelength_range, wavelength_type='Air', wavelength_accuracy=20, element_spectrum='C II-IV')
    <Table rows=3 names=('LAMBDA VAC ANG','SPECTRUM','TT','TERM','J J','LEVEL ENERGY  CM 1')>
    array([(196.8874, 'C IV', 'E1', '2S-2Po', '1/2-*', '0.00 -   507904.40'),
           (197.7992, 'C IV', 'E1', '2S-2Po', '1/2-*', '0.00 -   505563.30'),
           (199.0122, 'C IV', 'E1', '2S-2Po', '1/2-*', '0.00 -   502481.80')],
          dtype=[('LAMBDA VAC ANG', '<f8'), ('SPECTRUM', 'S4'), ('TT', 'S2'), ('TERM', 'S6'), ('J J', 'S5'), ('LEVEL ENERGY  CM 1', 'S18')])

.. _source: http://www.pa.uky.edu/~peter/atomic/documentation.html
.. _AstroPy: http://astropy.readthedocs.org/en/stable/
