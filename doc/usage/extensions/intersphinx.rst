:mod:`sphinx.ext.intersphinx` -- Link to other projects' documentation
======================================================================

.. module:: sphinx.ext.intersphinx
   :synopsis: Link to other Sphinx documentation.

.. index:: pair: automatic; linking

.. versionadded:: 0.5

This extension can generate automatic links to the documentation of objects in
other projects.

Usage is simple: whenever Sphinx encounters a cross-reference that has no
matching target in the current documentation set, it looks for targets in the
documentation sets configured in :confval:`intersphinx_mapping`.  A reference
like ``:py:class:`zipfile.ZipFile``` can then link to the Python documentation
for the ZipFile class, without you having to specify where it is located
exactly.

When using the "new" format (see below), you can even force lookup in a foreign
set by prefixing the link target appropriately.  A link like ``:ref:`comparison
manual <python:comparisons>``` will then link to the label "comparisons" in the
doc set "python", if it exists.

Behind the scenes, this works as follows:

* Each Sphinx HTML build creates a file named :file:`objects.inv` that contains
  a mapping from object names to URIs relative to the HTML set's root.

* Projects using the Intersphinx extension can specify the location of such
  mapping files in the :confval:`intersphinx_mapping` config value.  The mapping
  will then be used to resolve otherwise missing references to objects into
  links to the other documentation.

* By default, the mapping file is assumed to be at the same location as the rest
  of the documentation; however, the location of the mapping file can also be
  specified individually, e.g. if the docs should be buildable without Internet
  access.


Configuration
-------------

To use Intersphinx linking, add ``'sphinx.ext.intersphinx'`` to your
:confval:`extensions` config value, and use these config values to activate
linking:

.. confval:: intersphinx_mapping

   This config value contains the locations and names of other projects that
   should be linked to in this documentation.

   Relative local paths for target locations are taken as relative to the base
   of the built documentation, while relative local paths for inventory
   locations are taken as relative to the source directory.

   When fetching remote inventory files, proxy settings will be read from
   the ``$HTTP_PROXY`` environment variable.

   **Old format for this config value**

   This is the format used before Sphinx 1.0.  It is still recognized.

   A dictionary mapping URIs to either ``None`` or an URI.  The keys are the
   base URI of the foreign Sphinx documentation sets and can be local paths or
   HTTP URIs.  The values indicate where the inventory file can be found: they
   can be ``None`` (at the same location as the base URI) or another local or
   HTTP URI.

   **New format for this config value**

   .. versionadded:: 1.0

   A dictionary mapping unique identifiers to a tuple ``(target, inventory)``.
   Each ``target`` is the base URI of a foreign Sphinx documentation set and can
   be a local path or an HTTP URI.  The ``inventory`` indicates where the
   inventory file can be found: it can be ``None`` (an :file:`objects.inv` file
   at the same location as the base URI) or another local file path or a full
   HTTP URI to an inventory file.

   The unique identifier can be used to prefix cross-reference targets, so that
   it is clear which intersphinx set the target belongs to.  A link like
   ``:ref:`comparison manual <python:comparisons>``` will link to the label
   "comparisons" in the doc set "python", if it exists.

   **Example**

   To add links to modules and objects in the Python standard library
   documentation, use::

      intersphinx_mapping = {'python': ('https://docs.python.org/3', None)}

   This will download the corresponding :file:`objects.inv` file from the
   Internet and generate links to the pages under the given URI.  The downloaded
   inventory is cached in the Sphinx environment, so it must be re-downloaded
   whenever you do a full rebuild.

   A second example, showing the meaning of a non-``None`` value of the second
   tuple item::

      intersphinx_mapping = {'python': ('https://docs.python.org/3',
                                        'python-inv.txt')}

   This will read the inventory from :file:`python-inv.txt` in the source
   directory, but still generate links to the pages under
   ``https://docs.python.org/3``.  It is up to you to update the inventory file
   as new objects are added to the Python documentation.

   **Multiple targets for the inventory**

   .. versionadded:: 1.3

   Alternative files can be specified for each inventory. One can give a
   tuple for the second inventory tuple item as shown in the following
   example. This will read the inventory iterating through the (second)
   tuple items until the first successful fetch. The primary use case for
   this to specify mirror sites for server downtime of the primary
   inventory::

      intersphinx_mapping = {'python': ('https://docs.python.org/3',
                                        (None, 'python-inv.txt'))}

   For a set of books edited and tested locally and then published
   together, it could be helpful to try a local inventory file first,
   to check references before publication::

      intersphinx_mapping = {
          'otherbook':
              ('https://myproj.readthedocs.io/projects/otherbook/en/latest',
                  ('../../otherbook/build/html/objects.inv', None)),
      }

.. confval:: intersphinx_cache_limit

   The maximum number of days to cache remote inventories.  The default is
   ``5``, meaning five days.  Set this to a negative value to cache inventories
   for unlimited time.

.. confval:: intersphinx_timeout

   The number of seconds for timeout.  The default is ``None``, meaning do not
   timeout.

   .. note::

      timeout is not a time limit on the entire response download; rather, an
      exception is raised if the server has not issued a response for timeout
      seconds.


Showing all links of an Intersphinx mapping file
------------------------------------------------

To show all Intersphinx links and their targets of an Intersphinx mapping file,
run ``python -msphinx.ext.intersphinx url-or-path``.  This is helpful when
searching for the root cause of a broken Intersphinx link in a documentation
project. The following example prints the Intersphinx mapping of the Python 3
documentation::

   $ python -m sphinx.ext.intersphinx https://docs.python.org/3/objects.inv

Using Intersphinx with inventory file under Basic Authorization
---------------------------------------------------------------

Intersphinx supports Basic Authorization like this::

      intersphinx_mapping = {'python': ('https://user:password@docs.python.org/3',
                                        None)}

The user and password will be stripped from the URL when generating the links.
