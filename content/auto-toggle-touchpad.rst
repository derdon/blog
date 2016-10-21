Automatically enable and diable touchpad when (dis-)connecting USB mouse via udev
=================================================================================
:date: 2016-10-21
:category: linux
:status: draft
:summary: 

Motivation
----------
I own a laptop with a big touchpad. When an external mouse is plugged in,
I disable the touchpad so that I don't use it accidentally (e.g. while
typing or gaming). When taking the laptop with me, I don't also carry my
mouse but use the touchpad. Which means I have to enable it again. These
cumbersome steps take some time I would like to automate.

Introduction
------------
udev_

.. technology: udev rules; what is udev?


Step #1: Toggle touchpad manually from the terminal
---------------------------------------------------
Make shell aliases for enabling and disabling the touchpad (optional)


The tool ``xinput`` lets you 

::

    xinput list

::

    ⎡ Virtual core pointer                          id=2    [master pointer  (3)]
    ⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
    ⎜   ↳ TPPS/2 IBM TrackPoint                     id=15   [slave  pointer  (2)]
    ⎜   ↳ SynPS/2 Synaptics TouchPad                id=14   [slave  pointer  (2)]
    ⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
        ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
        ↳ Power Button                              id=6    [slave  keyboard (3)]
        ↳ Video Bus                                 id=7    [slave  keyboard (3)]
        ↳ Sleep Button                              id=8    [slave  keyboard (3)]
        ↳ AT Translated Set 2 keyboard              id=13   [slave  keyboard (3)]
        ↳ ThinkPad Extra Buttons                    id=16   [slave  keyboard (3)]
        ↳ Integrated Camera                         id=9    [slave  keyboard (3)]


::

    alias enable_touchpad='xinput set-prop 14 "Synaptics Off" 0'
    alias disable_touchpad='xinput set-prop 14 "Synaptics Off" 1'

Step #2: 


.. _udev: https://www.freedesktop.org/software/systemd/man/udev.html
