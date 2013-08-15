Week 10 — Extracting waveunit from FITS files
=============================================
:date: 2013-08-15
:category: gsoc
:summary: This week I worked on a seperate git branch to submit a pull
          request. I added a feature to extract the wavelength unit being
          used from a FITS header.

Motivation
----------
The ``query`` method of the class ``Database`` already works with
``Wave(100, 200, 'angstrom')``, but not as the user might expect it.
Currently, querying for the wave instance shown above will only search for
database entries with the saved wavelength unit *angstrom* and then check
if the stored wavelength is between 100 and 200 (inclusive). The goal is
to check every single database entry and convert units, if necessary.

Getting the waveunit from a FITS file
-------------------------------------
To get the wavelength unit from a FITS file, the first step that has to be
performed is to get a list of FITS headers using the function
``sunpy.io.fits.get_header``. Do not get confused by the name; the
function returns a list of headers, not just one header. My new function
``extract_waveunit`` receives one header and attempts several ways to find
the value of the waveunit. One goal of this function is to return strings
which can be used easily to make ``astropy.unit`` instances. Here is an
example of how to use it:

.. pycon::

    >>> from sunpy.io import fits
    >>> from sunpy.data.test.waveunit import MEDN_IMAGE
    >>> from astropy import units
    >>> header = fits.get_header(MEDN_IMAGE)[0]
    >>> waveunit = fits.extract_waveunit(header)
    >>> waveunit
    'nm'
    >>> getattr(units, waveunit)
    Unit("nm")
