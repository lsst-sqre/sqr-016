..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1
.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. note::

   **This technote is not yet published.**

   SQuaRE instructions for making official releases



Tokens
------

+-------------------------+----------------------------------------------------------+
| tag (release version)   | 13.0                                                     |
+-------------------------+----------------------------------------------------------+
| EUPS_PKGROOT filesystem | lsst-dev01.ncsa.illinois.edu:/lsst/distserver/production |
+-------------------------+----------------------------------------------------------+


Preparing for a candidate release
---------------------------------

Get codekit installed
^^^^^^^^^^^^^^^^^^^^^

If you are working interactively you want to set the environment variable DM_SQUARE_DEBUG to 1 as it gives more insight on what is going on with the Github REST API calls. 

Release the demo
^^^^^^^^^^^^^^^^

Go to https://github.com/lsst/lsst_dm_stack_demo/releases and 'Draft a
new release' with your tag. See its release history for examples.

Branching the docs
^^^^^^^^^^^^^^^^^^

At this point you should branch ``lsst/pipelines_lsst_io`` 

.. code-block:: bash

   git clone https://github.com/lsst/pipelines_lsst_io.git
   git checkout -b 13.0

Update the ``demo.rst`` page to point to the demo release you just made and use this version for testing your candidates as described in the pipelines documentation for testing your installation.


Candidate release
-----------------


Final source release
--------------------



Branching lsst
^^^^^^^^^^^^^^^

In this process we make use of the fact that git doesn't care whether
a ref is a tag or a branch to constrain the number of branches to
repositories that need retroactive maintainance or need to be
available in more than one cadence. One such example is the ``lsst``
repo since it containes ``newinstall.sh`` which sets the version of
eups, and that may be different for an official release than the
current bleed. 

The first repo that should be branched is lsst/lsst:

.. code-block:: bash

   git clone https://github.com/lsst/lsst.git
   git checkout -b 13.0
   
Now in ``lsst/scripts/newinstall.sh`` change the canonical reference for this newinstall to be one associated with the current branch::

  NEWINSTALL="https://raw.githubusercontent.com/lsst/lsst/13.0/scripts/newinstall.sh"

and commit and push.
  
This means that if you need to update newinstall.sh for bleed users, official-release users will not be prompted to update to the latest version, but will phone home against their official-release branch for hotfixes.

Also double-check for other things that might need to be updated, like the documentation links (though these should really be fixed on master prior to branching or cherry-picked back).

Doc update: newinstall.rst
^^^^^^^^^^^^^^^^^^^^^^^^^^

Update the ``newinstall.rst`` page on your release branch of
pipelines_lsst_io with the new download location of the newinstall.sh
script.



Final tag
^^^^^^^^^

Now it's time to lay down the final git tag. For repositories that
have already been branched with the 13.0 ref, that will fail, which is
fine.

This is mostly a repeat of the process for laying down the candidate tag but this time we use numeric tags so that eups will see them::

  # tag repos involved in the final candidate and final build
  github-tag-version --org lsst --team 'Data Management' --candidate v13_0_rc1 13.0 b2748
  github-tag-version --org lsst --team 'Database' --candidate v13_0_rc1 13.0 b2748

  # for externals NOTE THE v PREFIX to avoid stomping on the eups semantic versioning
  github-tag-version --org lsst --team 'DM Externals' --candidate v13_0_rc1 v13.0 b2748

Release build
^^^^^^^^^^^^^

- Submit the run-rebuild job with your parameters (eg. 13.0 v13.0)

- At this point you should not be seeing master-g type references as eups versions. Everything should have a tag-derviced version such as 13.0 if they are a DM repo and their semantic tag if they are external. If you see one, chase down why.
  
- Note your final bNNNN number for the publish

- Submit the run-publish job making sure you have selected 'package'' and not 'git' on the dropdown

  


