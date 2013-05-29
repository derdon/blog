using a custom pygments style via vim color schemes
===================================================
:date: 2011-05-16
:category: python
:summary: Vim color schemes define how syntax highlighting is done when
    editing files in (g)vim. This article explains how to use such a vim color
    scheme for code blocks in articles which use the library pygments_ to
    highlight code.

To convert a vim color schemes to python code, which is
required for later use, there is the script vim2pygments.py_ in the
pygments repository on bitbucket. You pass the path of the vim file to
this script. Example::

    ~/.vim/colors$ vim2pygments.py summerfruit.vim > styles.py

The generated python code is now saved in styles.py. Let's take a look at
it!

.. code-block:: py

    ~/.vim/colors$ cat styles.py
    # -*- coding: utf-8 -*-
    """
        Summerfruit Colorscheme
        ~~~~~~~~~~~~~~~~~~~~~~~
    
        Converted by Vim Colorscheme Converter
    """
    from pygments.style import Style
    from pygments.token import Token, Comment, Name, Keyword, Generic, Number, Operator, String
    
    class SummerfruitStyle(Style):
    
        background_color = '#ffffff'
        styles = {
            Token:              '#000000 bg:#ffffff',
            Generic.Output:     '#438ec3 bg:#b7dce8',
            Keyword:            '#fb660a bold',
            Number:             '#0086f7 bold',
            Name.Tag:           '#fb660a bold',
            String:             '#0086d2',
            Generic.Error:      '#ffffff bg:#d40000',
            Comment:            '#22a21f bg:#dbf3cd italic',
            Name.Attribute:     '#ff0086 bold',
            Name.Entity:        '#fd8900',
            Keyword.Type:       '#70796b bold',
            Name.Label:         '#ff0086',
            Name.Function:      '#ff0086 bold',
            Name.Variable:      '#ff0086 bold',
            Generic.Subheading: '#000000 bold',
            Generic.Heading:    '#000000 bold',
            Name.Constant:      '#0086d2',
            Comment.Preproc:    '#ff0007 bold',
        }

Now we open a new python shell and execute the following commands to
create a CSS file with the information gathered from the generated python
file:

.. code-block:: pycon

    >>> from pygments.formatters import HtmlFormatter
    >>> from styles import SummerfruitStyle
    >>> css_text = HtmlFormatter(style=SummerfruitStyle).get_style_defs()
    >>> with open('summerfruit.css', 'w') as cssfile:
    ...     cssfile.write(css_text)

We create a new HtmlFormatter because the wanted output is (X)HTML and
pass as the style the class we imported from the generated python file. To
get the CSS from a HtmlFormatter instance, the method ``get_style_defs``
is used. Finally, this string is written to a new CSS file. Now you can use
the CSS file in your pygments highlighted code by simply including it in a
way that the generated HTML file accesses it, e.g.:

.. code-block:: html

    <link rel="stylesheet" href="summerfruit.css" type="text/css">

.. _pelican: http://docs.notmyidea.org/alexis/pelican/
.. _pygments: http://pygments.pocoo.org
.. _vim2pygments.py: https://bitbucket.org/birkenfeld/pygments-main/src/4e66a4c4b3ed/scripts/vim2pygments.py
