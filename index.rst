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

Next, start a `community.lsst.org <clo>`_ post to use for status updates using
the clo-stubb_.

Branching the docs
^^^^^^^^^^^^^^^^^^

At this point you should branch `lsst/pipelines_lsst_io <pipelines_lsst_io>`_
so as not to capture any changes on the ``master`` branch that may occur during
the release process.

.. code-block:: bash

   git clone https://github.com/lsst/pipelines_lsst_io.git
   git checkout -b 14.0

Identify base weekly build
^^^^^^^^^^^^^^^^^^^^^^^^^^

Identify the *git tag* of the weekly build you wish to base the release release
candidate upon, say ``w.1999.42``.  This should be determined by discussion
with the product owner and team developer.


1st Release Candidate
---------------------

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins `release/official-release <official-release>`_ job.  The source
git refs should be only the tag of the "seed" weekly release.  The release tag
**must** start with a ``v`` and end with ``.rc1``.

See git-tags_ for details on the formatting of git tags.

Example:

.. code-block:: text

   SOURCE_GIT_REFS: w.1999.42
   RELEASE_GIT_TAG: v42.0.0.rc1
   O_LATEST: false

Announce rc1
^^^^^^^^^^^^

Update the release status `community.lsst.org <clo>`_ post to to announce the
availability of ``rc1``.


2nd+ Release Candidate(s)
-------------------------

An ``.rcX``, where X is ``> 1``, is only required if a problem is found with
the initial ``rc``.

**Any subsequent ``rc` differs slightly from the initial ``rc1`` process
because it inherently is not identical to a previous ``git tag`` (if it was,
there would be no reason to produce another ``rc``). The creation of a git
release branch prior to ``rc1`` would eliminate the differences.**

Branch, Merge, Tag
^^^^^^^^^^^^^^^^^^

Any git repository that needs to be modified for additional ``rc`` releases
should be **branched** and have the necessary changes merged to a release
branch.  Eg., if changes were needed in ``v42.0.0.rc1`` a "release branch"
along the lines of ``v42.x`` should be created in the repos that need changes.

(**TBD**: merge to master and cherry-pick to release branch or merge to release
branch and merge to ``master``???)

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins `release/official-release <official-release>`_ job.

For input source git refs use the previous ``rc`` tag along with the release
branch(es)t.  **Ensure** that the release branch is specified to the **left** of
the ``rcX`` tag in the listing of git refs.

Example 1:

.. code-block:: text

   SOURCE_GIT_REFS: v42.x v42.0.0.rc1
   RELEASE_GIT_TAG: v42.0.0.rc2
   O_LATEST: false

Example 2:

.. code-block:: text

   SOURCE_GIT_REFS: v42.x v42.0.0.rc5
   RELEASE_GIT_TAG: v42.0.0.rc6
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

As consequence of this behavior is that the final git tag **must** be present
prior to the production of ``eupspkg``/*eups tag*.

Build and Publish
^^^^^^^^^^^^^^^^^

Run the jenkins `release/official-release <official-release>`_ job.

The source input **must only be** the latest ``rc`` tag.

**The ``O_LATEST`` flag controls if the produced science pipelines docker image
has the ``-o_latest`` docker tags applied to it.  This should only be set on a
final release AND only if the release is the highest version release.  For
example, if ``99.0.0`` has been release and a ``98.0.1`` bugfix release is
being made, ``O_LATEST`` should not be set.**

Example 1:

.. code-block:: text

   SOURCE_GIT_REFS: v42.0.0.rc6
   RELEASE_GIT_TAG: 42.0.0
   O_LATEST: false

Branch ``newinstall.sh`` repo
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this process we make use of the fact that git doesn't care whether a ref is
a tag or a branch to constrain the number of branches to repositories that need
retroactive maintenance or need to be available in more than one cadence. One
such example is the ``lsst`` repo since it contains newinstall.sh_ which
sets the version of eups, and that may be different for an official release
than the current bleed.

Branch `lsst/lsst <lsst>`_:

.. code-block:: bash

   git clone https://github.com/lsst/lsst.git
   git checkout -b 42.0.0

Now in ``lsst/scripts/newinstall.sh`` change the canonical reference for this
newinstall to be one associated with the current branch:

.. code-block:: text

   NEWINSTALL_URL="https://raw.githubusercontent.com/lsst/lsst/14.0/scripts/newinstall.sh"

and commit and push.

This means that if you need to update ``newinstall.sh`` for bleed users,
official-release users will not be prompted to update to the latest version,
but will phone home against their official-release branch for hotfixes.

Also double-check for other things that might need to be updated, like the
documentation links (though these should really be fixed on master prior to
branching or cherry-picked back).

Documentation
^^^^^^^^^^^^^

Documentation to be collected for the release notes in pipelines_lsst_io_ is:

- Release notes from the T/CAMs for Pipelines, SUI, and DAX

- Characterization report from the DM or SQuaRE scientist

- Known issues and pre-requisites from the T/CAM for SQuaRE

- Before merging to master, ask the Documentation Engineer to review

- Update the ``newinstall.rst`` page on your release branch of
  pipelines_lsst_io_ with the new download location of the ``newinstall.sh``
  script.

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

  Summary
  -----------

  Release is complete

  Precursor Steps
  ---------------------------------

  1. Identify any pre-release blockers ("must-have features") :tools:
  2. Wait for them to clear


  Release Engineering Steps
  -------------------------------

  1. Eups publish rc1 candidate (based on b2748) (also w_2017_33)
  1. Git Tag v14.0.rc1
  1. Branch v14 of newinstall.sh
  1. **Wait for first round of bugs to clear**
  1.Repeat last 2 steps, .rcN candidates  <-- final candidate is rc1 [yay!]
  1. Confirm DM Externals are at stable tags
  1. Tag DM Auxilliary (non-lsst_distrib) repos
  1. Full OS testing (see https://ls.st/faq )
  1. Git Tag 14.0, rebuild, eups publish

  Binary release steps
  ------------------------

  1. Produce factory binaries
  1. Test factory binaries
  1. Gather contributed binaries

  Documentation Steps
  -------------------------

  1. Update Prereqs/Install
  1. Update Known Issues
  1. Gather Release notes
  1. Gather Metrics report
  1. **Email announcement**



Github teams
============

There are three "special" teams in the LSST Github org:

- ``Data Management``

- ``DM Externals``

- ``DM Auxilliaries``

These are used in the release process in the following way:

- ``Data Management`` repos are a dependency of ``lsst_distrib`` and should be
  tagged with the bare release version, eg. ``14.0``, unless the repo is also a
  member of the ``DM Externals`` team.  All repos tagged as part of a release
  should be members of the ``Data Management`` team to ensure that DM
  developers are able to modify all components of a release.

- ``DM Externals`` also indicates a dependency of ``lsst_distrib`` but one that
  is tagged with a ``v`` prefix in front of the release version. Eg., ``v14.0``
  This is required because ``lsst-build`` derives the eups product version
  string from git tags that begin with a number.  DM developers prefer that
  eups display external packages version string rather than of a DM composite
  release. Thus the ``v`` prefix causes the git tag to be ignored by
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
  on DM produced code. Eg., ``42.0.0 -> v42.0.0``

- Non-"official" releases, release candidates, weekly builds, etc. **must**
  start with a *letter*

- **shall** only use ``[a-z]``, ``[0-9]``, and ``.``

  * *lowercase* latin alphabet characters **shall** be used; *uppercase*
    characters are forbidden

  * These common characters **must not** be used: ``-``, ``_``, ``/``


Examples of *valid* (good) git tags

.. code-block:: text

  # unofficial builds
  d.2038.01.19
  w.2038.03

  # release candidate
  v42.0.0.rc99

  # official release of DM produced code
  42.0.0

  # official release of external/third-party product
  v42.0.0

Examples of *invalid* (bad) git tags

.. code-block:: text

  d_2038_01_19
  w_2038_03
  v42-0-0-rc99
  42_0_0
  v42_0_0
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
  d_2038_01_19
  w_2038_03

  # release candidate
  v42_0_0_rc99

  # official release of DM produced code AND external/third-party product
  v42_0_0

Examples of *invalid* (bad) eup tags

.. code-block:: text

  123
  d.2038.01.19
  w.2038.03
  v42_0_0-rc99
  42.0.0
  v42.0.0
  foo/bar

git <-> eups tag conversion
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The "tags" along each row in the following table should be considered
equivalent conversions.

============  ============  ========
internal git  external git  eups tag
============  ============  ========
d.2038.01.19  d.2038.01.19  d_2038_01_19
w.2038.03     w.2038.03     w_2038_03
v42.0.0.rc99  v42.0.0.rc99  v42_0_0_rc99
42.0.0        v42.0.0       v42_0_0
============  ============  ========



Conda Environment/Packages Update
=================================

There are conflicting pressures of updating the conda package list frequently
to minimize the amount of [likely] breakage at one time and resisting changes
as the git ``sha1`` of the conda environment files is used to defined the
``ABI`` of the eups ``tarball`` packages.


Adding a new Conda package
--------------------------

#. The name of the package needs to "bleed" or un-versioned environment files in
   the ``lsst/lsstsw`` repo. Which are:

    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_bleed-linux-64.txt
    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_bleed-osx-64.txt

    These env files are currently kept in the original conda environment file
    format and have not yet been migrated to the newer ``yaml`` based format as
    it only works with fairly recent conda releases. (*TODO* migrate to `yaml`
    format after DM-14011 is merged).

    The bleed env files should be keep in sync with the *exception* of the
    ``nomkl`` package, which is required on ``linux``.  Also note that the env
    files should be kept sorted to allow for clean ``diff`` s.

#. The regular conda env files need to be updated by running a fresh install
   with ``deploy -b``` (bleed install) and then manually exporting the env to a
   file.  A side effect of this is other package versions will almost certainly
   change and this **is an ABI breaking event**. The existing env files are:

    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_packages-linux-64.txt
    - https://github.com/lsst/lsstsw/blob/master/etc/conda3_packages-osx-64.txt

    ``conda list -e`` should be run on ``linux`` and ``osx`` installs and the
    results committed for both platforms as **a single commit** so that the the
    abbrev sha1 of the latest commit for both files will be the same.

#. As an abbreviated sha1 of the ``lsst/lsstsw`` repo is used to select which
   [version of] conda env files are used and to define the eups binary tarball
   "ABI", jenkins needs to know this value to ensure that ``newinstall.sh`` is
   explicitly using the correct ref and to construct the paths of the tarball
   ``EUPS_PKGROOT`` s.  The ``lsstsw_ref`` / ``LSST_LSSTSW_REF`` needs to be
   updated at:

    - https://github.com/lsst-sqre/jenkins-dm-jobs/blob/master/etc/scipipe/build_matrix.yaml#L10
    - https://github.com/lsst/lsst/blob/master/scripts/newinstall.sh#L33

#. The ~last major release should be rebuilt in the new "ABI" ``EUPS_PKGROOT`` so
   that that newinstall.sh from master will still be able to do a binary
   install of the current major release.  This may be done by triggering a
   Jenkins ``release/tarball-matrix`` build.



Making a Manual Release
=======================

**The official release process is captured as a jenkins job:
https://ci.lsst.codes/job/release/job/official-release/ which should be
preferred over manually enumerating all of the release steps.**


Get codekit installed
---------------------

The release manager can follow the instructions at:

https://github.com/lsst-sqre/sqre-codekit

If you are working interactively you want to set the environment variable
``DM_SQUARE_DEBUG`` to ``1`` as it gives more insight on what is going on with
the Github REST API calls.


1st Release Candidate
---------------------

The following sections attempt to document the individual steps that
``official-release`` is essentially composed of.

Identify base weekly build
^^^^^^^^^^^^^^^^^^^^^^^^^^

Identify the *git tag* of the weekly build you wish to base the release
release candidate upon, say ``w.2017.33``.

Look into the text of the annotated tag (using git log or the Github UI) to see
what the manifest ID (``bXXXX``) number was or use git cli on a local clone of
a git repo known to be part of the weekly build.

.. code-block:: bash

    # find manifest ID of a previous weekly build

    $ git clone https://github.com/lsst/base
    Cloning into 'base'...
    remote: Counting objects: 1041, done.
    remote: Compressing objects: 100% (9/9), done.
    remote: Total 1041 (delta 6), reused 0 (delta 0), pack-reused 1032
    Receiving objects: 100% (1041/1041), 259.28 KiB | 1.21 MiB/s, done.
    Resolving deltas: 100% (521/521), done.
    $ cd base
    $ git tag w.2017.33 -n1
    w.2017.33       Version w.2017.33 release from w.2017.33/b2999

In the above example, the manifest ID is ``b29999``.

*git tag* the candidate
^^^^^^^^^^^^^^^^^^^^^^^

Tagging *first* and *third* parties repos allows the release to be reproducible in
the future and is necessary for the final build process.

Note that the difference in *git tag* name convention between first and third
parties is automatically handled by the ``--external-team`` flag.

.. code-block:: bash

   # create git "release candidate" tag from manifest ID

   github-tag-version \
     --debug \
     --token **** \
     --user sqreadmin \
     --email sqre-admin@lists.lsst.org \
     --org lsst \
     --allow-team 'Data Management' \
     --allow-team 'DM Externals' \
     --external-team 'DM Externals' \
     --deny-team 'DM Auxilliaries' \
     --manifest-only \
     --manifest b2999 \
     v42.0.0.rc1

.. code-block:: bash

    # create git on aux repos
    # previous weekly tag.

    github-tag-teams \
      --debug \
      --token **** \
      --user sqreadmin \
      --email sqre-admin@lists.lsst.org \
      --org lsst \
      --allow-team 'DM Auxilliaries' \
      --deny-team 'DM Externals' \
      --ignore-existing-tag \
      --tag v42.0.0.rc1

**XXX this is currently broken in that the git tag will be placed at the
current HEAD of the default branch instead of at the same location as the**

Build and Publish eups ``eupspkg`` packages + eups tag
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

https://ci.lsst.codes/blue/organizations/jenkins/release%2Frun-rebuild/activity

The resulting manifest ID needs to be retrieved to use as input for subsequent
jobs.

.. code-block:: text

    REFS: v42.0.0.rc1
    PRODUCTS: lsst_distrib
    BUILD_DOCS: true

https://ci.lsst.codes/blue/organizations/jenkins/release%2Frun-publish/activity

.. code-block:: text

    PRODUCTS: lsst_distrib
    EUPSPKG_SOURCE: git
    TAG: v42_0_0_rc1
    BUILD_ID: bXXXX

Build and Publish eups ``tarball`` packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Note that the jenkins ``release/official-release`` job does not trigger
``release/tarball-matrix`` and triggers ``release/tarball`` build(s) directly
so as to have more explicit control over the parameters.

https://ci.lsst.codes/blue/organizations/jenkins/release%2Ftarball-matrix/activity

.. code-block:: text

    PRODUCTS: lsst_distrib
    EUPS_TAG: v42_0_0_rc1
    SMOKE: true
    RUN_SCONS_CHECK: true
    PUBLISH: true

Build and Publish ``scipipe`` docker image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

https://ci.lsst.codes/blue/organizations/jenkins/release%2Fdocker%2Fbuild-stack/activity

.. code-block:: text

    PRODUCTS: lsst_distrib
    TAG: v42_0_0_rc1  # eups tag
    NO_PUSH: false

Build and Publish ``jupyterlab`` docker image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

https://ci.lsst.codes/blue/organizations/jenkins/sqre%2Finfra%2Fbuild-jupyterlabdemo/activity

.. code-block:: text

    WIPEOUT: true
    MANIFEST_ID: bXXXX
    COMPILER: devtoolset-6
    EUPS_TAG: v42_0_0_rc1
    RELEASE_IMAGE: lsstsqre/centos:7-stack-lsst_distrib-v42_0_0_rc1
    NO_PUSH: false

Run ``validate_drp``
^^^^^^^^^^^^^^^^^^^^

https://ci.lsst.codes/blue/organizations/jenkins/sqre%2Fvalidate_drp/activity

.. code-block:: text

    WIPEOUT: true
    MANIFEST_ID: bXXXX
    COMPILER: devtoolset-6
    EUPS_TAG: v42_0_0_rc1
    RELEASE_IMAGE: lsstsqre/centos:7-stack-lsst_distrib-v42_0_0_rc1


2nd+ Release Candidate(s)
-------------------------

Produce new manifest (``manifest ID``)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use the previous ``rc`` tag along with the release branch to produce a new
manifest.

https://ci.lsst.codes/blue/organizations/jenkins/release%2Frun-rebuild/activity

The resulting manifest ID needs to be retrieved to use as input for subsequent
jobs.

.. code-block:: text

    REFS: v42.x v42.0.0.rc1
    PRODUCTS: lsst_distrib
    BUILD_DOCS: false
    PREP_ONLY: true


Final Release
-------------

Final tag
^^^^^^^^^

XXX failures are now fatal...

Now it's time to lay down the final git tag. For repositories that have already
been branched with the ``14.0`` ref, that will fail, which is fine.

This is mostly a repeat of the process for laying down the candidate tag but
this time we use numeric tags so that eups will see them:

.. code-block:: bash

   # tag repos involved in the final candidate as the final build
   github-tag-version \
     --org lsst \
     --allow-team 'Data Management' \
     --allow-team 'DM Externals' \
     --external-team 'DM Externals' \
     --deny-team 'DM Auxilliaries' \
     --debug \
     --candidate 'v14_0_rc2 \
     '14.0' 'b3176'

Release build
^^^^^^^^^^^^^

- Submit the run-rebuild job with your parameters (eg. ``14.0`` ``v14.0``)

- At this point you should not be seeing master-g type references as eups
  versions. Everything should have a tag-derived version such as ``14.0`` if
  they are a DM repo and their semantic tag (eg. ``pyfits 3.0``) if they are
  external.  If you see one, you need to chase down why. The only situation
  that should happen is if a third party but a branch is used for LSST
  development that lacks any other type of semantic versioning (in the ``14.0``
  release this included starlink_ast and jointcal_cholmod.

- Note your final ``bNNNN`` number for the publish (either from the build log
  or by looking at the next of the annotated ``14.0`` tag on any repo eg. afw).

- Submit the run-publish job making sure you have selected ``package`` and not
  ``git`` as the option.


.. _official-release: https://ci.lsst.codes/blue/organizations/jenkins/release%2Fofficial-release/activity/
.. _pipelines_lsst_io: https://github.com/lsst/pipelines_lsst_io
.. _clo: https://community.lsst.org
.. _lsst: https://github.com/lsst/lsst
.. _newinstall.sh: https://github.com/lsst/lsst/blob/master/scripts/newinstall.sh
