######################
Stack release playbook
######################

.. abstract::

   SQuaRE instructions for making official releases

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




.. Add content below. Do not include the document title.

.. note::

   SQuaRE instructions for making official releases



The LSST Science Pipelines Release Process
==========================================

The LSST stack is a large mixed Python/C++ codebase split among many repos with
multiple dependencies between them built by a niche build system. The release
process is therefore a bit complex and can be protracted. In an attempt not to
have to co-ordinate the state of the release with 80+ developers, or worse,
have them down tools while a release is being minted, this process has been
optimized to impact development as little as possible. Unless a developer needs
to be called upon to fix a failing integration or platform test, she can carry
on as normal.

In gross, the conceptual steps are:

#. A successful weekly release is identified that forms the basis for the
   official release.

#. The first release candidate (``rc1``) is created using that weekly build as a seed.
   This means that it does not matter if the codebase's master has moved on in
   the meanwhile.

#. The ``rc`` is announced for developer testing

#. Additional ``rc`` are created and announced, if/as necessary to resolve
   any release blocking issues that may arise.

#. Documentation is updated on a branch (release notes, installation
   instructions, metrics report)

#. The final release is created using the last ``rc`` verbatim (ie., the
   accepted ``rc`` and final release code should be identical except for the
   name of git/eups tags) and the documentation merged to ``master``.


Kicking-off the process
-----------------------

The first step is to establish whether it is a good time to make a release, eg.
if many developers are about to push significant features, it may be better to
wait for them to finish. Announce the intent to make a release in the ``#dm``
channel and inform all DM T/CAMs and the DM System Engineer.

If it is generally a good time, identify the nearest weekly release, this will
be the seed for the release candidate.

Announce start of release process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next, start a clo_ post to use for status updates using the clo-stubb_.

Branching the docs
^^^^^^^^^^^^^^^^^^

At this point you should branch pipelines_lsst_io_.  so as not to capture any
changes on the ``master`` branch that may occur during the release process.

Note that the branch does not have a ``v`` prefix.

.. code-block:: bash

   git clone https://github.com/lsst/pipelines_lsst_io.git
   cd pipelines_lsst_io
   git checkout -b 888.0.x
   git push -u origin 888.0.x

Identify base weekly build
^^^^^^^^^^^^^^^^^^^^^^^^^^

Identify the *git tag* of the weekly build you wish to base the release release
candidate upon, say ``w.9999.52``.  This should be determined by discussion
with the product owner and team developer.

Example method for listing weekly git tags:

.. code-block:: bash

   git clone https://github.com/lsst/lsst_distrib.git
   cd lsst_distrib
   git tag -l w.*


1st Release Candidate
---------------------

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins official-release_ job.  The source git refs should be only the
tag of the "seed" weekly release.  The release tag **must** start with a ``v``
and end with ``.rc1``.

See git-tags_ for details on the formatting of git tags.

Example:

.. code-block:: text

   SOURCE_GIT_REFS: w.9999.52
   RELEASE_GIT_TAG: v888.0.0.rc1
   O_LATEST: false

Announce rc1
^^^^^^^^^^^^

Update the release status clo_ post to to announce the
availability of ``rc1``.


2nd+ Release Candidate(s)
-------------------------

An ``.rcX``, where X is ``> 1``, is only required if a problem is found with
the initial ``rc``.

Any subsequent ``rc`` differs slightly from the initial ``rc1`` process
because it inherently is not identical to a previous ``git tag`` (if it was,
there would be no reason to produce another ``rc``). The creation of a git
release branch prior to ``rc1`` would eliminate the differences.

Branch, Merge
^^^^^^^^^^^^^

Any git repository that needs to be modified for additional ``rc`` releases
should be **branched** and have the necessary changes merged to a release
branch.  Eg., if changes were needed in ``v888.0.0.rc1`` a "release branch"
along the lines of ``v888.0.x`` should be created in the repos that need
changes.

Back ports and cherry-picking to release branches are described in https://developer.lsst.io/work/backports.html.

Test Release Branch(es)
^^^^^^^^^^^^^^^^^^^^^^^

As a sanity check, run the jenkins stack-os-matrix_ job to verify that the
release branch + previous ``rcX`` tag combination is buildable prior to
attempting to build and publish the new ``rc``.

Example 1:

.. code-block:: text

   REFS: v888.0.x v888.0.0.rc1
   PRODUCTS: lsst_distrib lsst_ci

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins official-release_ job.

For input source git refs use the previous ``rc`` tag along with the release
branch(es).  **Ensure** that the release branch is specified to the **left** of
the ``rcX`` tag in the listing of git refs.

Example 1:

.. code-block:: text

   SOURCE_GIT_REFS: v888.0.x v888.0.0.rc1
   RELEASE_GIT_TAG: v888.0.0.rc2
   O_LATEST: false

Example 2:

.. code-block:: text

   SOURCE_GIT_REFS: v888.0.x v888.0.0.rc5
   RELEASE_GIT_TAG: v888.0.0.rc6
   O_LATEST: false

Announce rcX
^^^^^^^^^^^^

Again, the post made from the clo-stubb_ should be updated to announce the
current ``rcX``.


Final Release
-------------

Note that a *Final Release* differs from a *Release Candidate* in that the DM
internal/first party git repositories receive a *git tag* that *does not* have
an alphabetic prefix (eg., ``v``).  This has the effect of changing the *eups*
version strings as ``lsst-build`` sets the *eups* product version based on the
most recent git ref that has an *integer* as the first character.

A consequence of this behavior is that the final git tag **must** be present
prior to the production of ``eupspkg``/*eups tag*.

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins official-release_ job.

The source input **must only be** the latest ``rc`` tag.

The ``O_LATEST`` flag controls if the produced science pipelines docker image
has the ``-o_latest`` docker tags applied to it.  This should only be set on a
final release AND only if the release is the highest version release.  For
example, if ``99.0.0`` has been release and a ``98.0.1`` bugfix release is
being made, ``O_LATEST`` should not be set.

Example 1:

.. code-block:: text

   SOURCE_GIT_REFS: v888.0.0.rc6
   RELEASE_GIT_TAG: 888.0.0
   O_LATEST: true

Documentation
^^^^^^^^^^^^^

Documentation to be collected for the release notes in pipelines_lsst_io_ is:

- Release notes from the T/CAMs for Pipelines, SUI, and DAX

- Characterization report from the DM or SQuaRE scientist

- Known issues and pre-requisites from the T/CAM for SQuaRE

- Before merging to master, ask the Documentation Engineer to review

Documenting Deprecations and Removals
"""""""""""""""""""""""""""""""""""""

Deprecated interfaces may be removed in the major release after the one in which their deprecations first appear.
These deprecations must be included in the release notes.

To identify all deprecations that have to be mentioned in a release note, we
search the codebase looking for specific strings. The application **ack** is
used here as a reference, since it is easy to install in Unix systems [#ack]_.

.. [#ack] The command **ack** may have to be installed separately.

These are the strings to search:

- **python deprecations**: ``ack -A 3 --python "@deprecated\(" stack/``

- **pybind11 deprecations**: ``ack -A 3 --python "deprecate_pybind11" stack/``

- **C++ deprecations**: ``ack -A 3 --cpp "\[deprecated\(" stack/``

- **config deprecations**: ``ack -B 3 --python "^\s+deprecated=" stack/``

Deprecated code should be removed in the major release following the one in which it was deprecated (though removal work is occassionally not scheduled in time for this to occur).
Removals should be documented in another section of the release notes.
The set of deprecations in the release notes for the previous release is probably the most complete list of code that *might* have been removed in a release, but each of these must be checked against Jira or the code itself to determine whether removal actually occurred.
Deprecations that are not followed by removals should be left in the release notes as deprecations.

Announce official release
^^^^^^^^^^^^^^^^^^^^^^^^^

Announce the final release on clo_.


Other OS checking
-----------------

While we only officially support the software on certain platforms
(`RHEL/CentOS 7` is the reference, and we CI `MacOS` and `RHEL 6`), we check in
a number of other popular platforms (eg `Ubuntu`, newer versions of `CentOS`
etc) by spinning up machines on Digital Ocean (typically) and following the
user install instructions. This also allows us to check the user from-scratch
installation instructions including the pre-requisites.



.. _clo-stubb:

c.l.o stubb
===========

.. code-block:: text

  Here is where we currently are in the release process. Current step in bold.

  Release Precursor Steps
  ---------------------------------

  1. Identify any pre-release blockers ("must-have features") :tools:
  Contributors check if there are outstanding issues that have to be included in the next release and relate them as blocker to the above issue DM-XXXXX.
  1. Wait untill all blocking issues are resolved.
  1. Create Jira issues for each release activity.
  1. Check that the weekly build is scientifically suitable to be used as starting point for the release

  Release Jira issue: https://ls.st/DM-XXXXX
  Tentative weekly to use as starting point for the release is w_20YY_WW
  Tentative target date to close the release is YYYY-MM-DD.

  Release Engineering Steps
  -------------------------------

  1. Create first release candidate vM.m.p.rc1
  1. Release candidate vM.m.p.rc1 available:
   - Build: bxxxxx
   - Weekly: w_20YY_WW
  1. Build the release candidate on supported platforms. Report bugs in Jira if any.
  1. Invite developers, contributors and downstram users to verify the release candidate and report bugs in Jira if any.
  1. Wait for bugs and additional issues to be identified, fixed and ported to the release branch.
  1. Create new release candidates if bugs / new issues have been fixed in the release branch
  1. Create official release M.m.p

  Documentation Steps
  -------------------------

  In parallel with the engineering steps after rc1 is available.
  [Integration on b.M.x branch of pipelines_lsst_io](https://github.com/lsst/pipelines_lsst_io/pull/TBD)

  1. Update Prereqs/Install
  1. Gather Release notes
  1. Gather Metrics report
  1. Update Known Issues
  1. Release availability community post



Github teams
============

There are three "special" teams in the LSST Github org:

- ``Data Management``

- ``DM Externals``

- ``DM Auxilliaries``

These are used in the release process in the following way:

- ``Data Management`` repos are a dependency of ``lsst_distrib`` and should be
  tagged with the bare release version, eg. ``888.0.0``, unless the repo is also
  a member of the ``DM Externals`` team.  All repos tagged as part of a release
  should be members of the ``Data Management`` team to ensure that DM
  developers are able to modify all components of a release.

- ``DM Externals`` also indicates a dependency of ``lsst_distrib`` but one that
  is tagged with a ``v`` prefix in front of the release version. Eg.,
  ``v888.0.0`` This is required because ``lsst-build`` derives the eups product
  version string from git tags that begin with a number.  DM developers prefer
  that eups display external packages version string rather than of a DM
  composite release. Thus the ``v`` prefix causes the git tag to be ignored by
  ``lsst_distrib``.  "External" repos must not also be members of ``DM
  Auxilliaries``.

- ``DM Auxilliaries`` are repos that we want to snapshot as part of a release
  but are not an eups dependency of ``lsst_distrib``. "Aux" repos must not also
  be members of ``DM Externals``.



Format of "tags"
================

.. _git-tags:

git tags
--------

- DM produced code this is part of an "official" release  **must** have a git
  tag that starts with a *number*

- "official" release git tags on external/third-party software that DM has
  repackaged must be prefixed with a ``v`` but are otherwise identical to that
  on DM produced code. Eg., ``888.0.0 -> v888.0.0``

- Non-"official" releases, release candidates, weekly builds, etc. **must**
  start with a *letter*

- **shall** only use ``[a-z]``, ``[0-9]``, and ``.``

  * *lowercase* latin alphabet characters **shall** be used; *uppercase*
    characters are forbidden

  * These common characters **must not** be used: ``-``, ``_``, ``/``


Examples of *valid* (good) git tags

.. code-block:: text

  # unofficial builds
  d.9999.01.02
  w.9999.52

  # release candidate
  v888.0.0.rc99

  # official release of DM produced code
  888.0.0

  # official release of external/third-party product
  v888.0.0

Examples of *invalid* (bad) git tags

.. code-block:: text

  d_9999_01_02
  w_9999_52
  v888-0-0-rc99
  888_0_0
  v888_0_0
  foo/bar

eups tags
^^^^^^^^^

- **must not** start with a numeric value

- **shall** only use ``[a-z]``, ``[0-9]``, and ``_``

  * *lowercase* latin alphabet characters **shall** be used; *uppercase*
    characters are forbidden

  * EUPS reportedly has or has had problems with ``.`` and ``-``

- official releases and release candidates **must** be prefixed with ``v``


Examples of *valid* (good) eups tags

.. code-block:: text

  # unofficial builds
  d_9999_01_02
  w_9999_52

  # release candidate
  v888_0_0_rc99

  # official release of DM produced code AND external/third-party product
  v888_0_0

Examples of *invalid* (bad) eup tags

.. code-block:: text

  123
  d.9999.01.02
  w.9999.52
  v888_0-rc99
  888.0.0
  v888.0.0
  foo/bar

git <-> eups tag conversion
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The "tags" along each row in the following table should be considered
equivalent conversions.

=============  ============  =============
internal git   external git  eups tag
=============  ============  =============
d.9999.01.02   d.9999.01.19  d_9999_01_02
w.9999.52      w.9999.52     w_9999_52
v888.0.0.rc99  v88.0.0.rc99  v888_0_0_rc99
888.0.0        v888.0.0      v888_0_0
=============  ============  =============



Conda Environment/Packages Update
=================================

Conda package dependencies are now maintained in the rubin-env meta packages.
For details see https://developer.lsst.io/stack/conda.html

.. _official-release: https://rubin-ci.slac.stanford.edu/blue/organizations/jenkins/release%2Fofficial-release/activity/
.. _pipelines_lsst_io: https://github.com/lsst/pipelines_lsst_io
.. _clo: https://community.lsst.org
.. _lsst: https://github.com/lsst/lsst
.. _stack-os-matrix: https://rubin-ci.slac.stanford.edu/blue/organizations/jenkins/stack-os-matrix/activity/
