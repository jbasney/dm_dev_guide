########################################################
Using Git LFS (Large File Storage) for data repositories
########################################################

This page describes how to use Git LFS for DM development.

DM uses Git LFS to manage test datasets within our :doc:`normal Git workflow </processes/workflow>`.
`Git LFS is developed by GitHub <https://git-lfs.github.com/>`_, though DM uses its own backend storage infrastructure (see `SQR-001: The Git LFS Architecture <http://sqr-001.lsst.io>`_ for background).

All DM repositories should use Git LFS to store binary data, such as FITS files, for :abbr:`CI (Continuous Integration)`.
Examples of LFS-backed repositories are `lsst/afw <https://github.com/lsst/afw>`_, `lsst/hsc_ci <https://github.com/lsst/ci_hsc>`_, `lsst/testdata_decam <https://github.com/lsst/testdata_decam>`_ and `lsst/testdata_cfht <https://github.com/lsst/testdata_cfht>`_.

**On this page**

- :ref:`git-lfs-install`
- :ref:`git-lfs-config`
- :ref:`git-lfs-auth`
- :ref:`git-lfs-using`
- :ref:`git-lfs-tracking`
- :ref:`git-lfs-create`

.. _git-lfs-install:

Installing Git LFS
==================

Download and install the :command:`git-lfs` client by visiting the `Git LFS <https://git-lfs.github.com>`_ homepage.
Many package managers, like Homebrew_ on the Mac, also provide :command:`git-lfs` (``brew install git-lfs`` for example).

We recommend using the latest Git LFS client.
The *minimum* usable client version for LSST is git-lfs v1.1.0.

Git LFS requires Git version 1.8.2 or later to be installed.

Before you can use Git LFS with LSST data you'll need to configure by following the next section.

.. _git-lfs-config:

Configuring Git LFS
===================

.. _git-lfs-basic-config:

Basic configuration
-------------------

After you've installed Git LFS, run:

.. code-block:: bash

   git lfs install

This is the regular Git LFS configuration step that adds a ``filter "lfs"`` section to :file:`~/.gitconfig`.
Additional configurations for LSST are next.

.. _git-lfs-config-lsst:

Configuration for LSST
----------------------

LSST uses its own Git LFS servers.
This section describes how to configure Git LFS to pull from LSST's servers.
If you are running an older client, version 1.2 or earlier, follow the note at the end of this section.

First, add these lines into your :file:`~/.gitconfig` file:

.. literalinclude:: snippets/git_lfs_gitconfig.txt
   :language: text

Then add these lines into your :file:`~/.git-credentials` files (create one, if necessary):

.. literalinclude:: snippets/git_lfs_git-credentials.txt
   :language: text

Trying cloning a small data repository to test your configuration:

.. code-block:: bash

   git clone https://github.com/lsst/testdata_decam

*That's it.*

.. note::

   **Configuration for Git LFS v1.2 and earlier***

   The legacy Git LFS client (versions *earlier* than 1.3) has two configuration differences compared to the modern configuration described above.
   
   First, add these lines into your :file:`~/.gitconfig` file:

   .. literalinclude:: snippets/git_lfs_gitconfig_legacy.txt
      :language: text

   Then add these lines into your :file:`~/.git-credentials` file (create one, if necessary):

   .. literalinclude:: snippets/git_lfs_git-credentials_legacy.txt
      :language: text
   
.. _git-lfs-auth:

Authenticating for push access
==============================

If you want to push to a LSST Git LFS-backed repository you'll need to configure and cache your credentials.

First, set up a credential helper to manage your GitHub credentials (Git LFS won't use your SSH keys).
:ref:`We describe how to set up a credential helper for your system in the Git set up guide <git-credential-helper>`.

Then the next time you run a Git command that requires authentication, Git will ask you to authenticate with LSST's Git LFS server::

   Username for 'https://git-lfs.lsst.codes': <GitHub username>
   Password for 'https://<git>@git-lfs.lsst.codes': <GitHub password or token>

At the prompts, enter your GitHub username and password.

Once your credentials are cached, you won't need to repeat this process on your system (:ref:`unless you opted for the cache-based credential helper <git-credential-helper>`).

.. note::

   **Working with GitHub Two Factor Authentication**

   If you have `GitHub's two-factor authentication <https://help.github.com/articles/about-two-factor-authentication/>`_ enabled, use a personal access token instead of a password.
   You can set up a personal token at https://github.com/settings/tokens with ``read:org`` permissions.

.. _git-lfs-using:

Using Git LFS-enabled repositories
==================================

Git LFS operates transparently to the user.
*Just use the repo as you normally would any other Git repo.*
All of the regular Git commands just work, whether you are working with LFS-managed files or not.

There are two caveats for working with LFS: HTTPS is always used, and Git LFS must be told to track new binary file types.

First, DM's LFS implementation mandates the HTTPS transport protocol.
Developers used to working with `ssh-agent <http://www.openbsd.org/cgi-bin/man.cgi?query=ssh-agent&sektion=1>`_ for passwordless GitHub interaction should use a :ref:`Git credential helper <git-credential-helper>`, and follow the :ref:`directions above <git-lfs-auth>` for configuring their credentials.

Note this *does not* preclude using ``git+git`` or ``git+ssh`` for working with a Git remote itself; it is only the LFS traffic that always uses HTTPS.

Second, in an LFS-backed repository, you need to specify what files are stored by LFS rather than regular Git storage.
You can run

.. code-block:: bash

   git lfs track

to see what file types are being tracked by LFS in your repository.
:ref:`We describe how to track additional file types below <git-lfs-tracking>`.

.. _git-lfs-tracking:

Tracking new file types
=======================

Only file types that are specifically *tracked* are stored in Git LFS rather than the standard Git storage.

To see what file types are already being tracked in a repository:

.. code-block:: bash

   git lfs track

To track a *new* file type (FITS files, for example):

.. code-block:: bash

   git lfs track "*.fits"

Git LFS stores information about tracked types in the :file:`.gitattributes` file.
This file is part of the repo and tracked by Git itself.

You can ``git add``, ``commit`` and do any other Git operations against these Git LFS-managed files.

To see what files are being managed by Git LFS, run:

.. code-block:: bash

   git lfs ls-files

.. _git-lfs-create:

Creating a new Git LFS-enabled repository
=========================================

Configuring a new Git repository to store files with DM's Git LFS is easy.
First, initialize the current directory as a repository:

.. code-block:: bash

   git init .

Make a file called :file:`.lfsconfig` *within the repository*, and write these lines into it:

.. code-block:: text

   [lfs]
        url = https://git-lfs.lsst.codes

Note that older versions of Git LFS used :file:`.gitconfig` rather than :file:`.lfsconfig`.
As of Git LFS version 1.1 `.gitconfig has been deprecated <https://github.com/github/git-lfs/pull/837>`_, but support will not be dropped until LFS version 2.

Next, track some files types.
For example, to have FITS and ``*.gz`` files tracked by Git LFS,

.. code-block:: bash

   git lfs track "*.fits"
   git lfs track "*.gz"

Add and commit the :file:`.lfsconfig` and :file:`.gitattributes` files to your repository.

You can then push the repo up to github with

.. code-block:: bash
   
   git remote add origin <remote repository URL>
   git push origin master

We also recommend that you include a link to this documentation page in your :file:`README` to help those who aren't familiar with DM's Git LFS.

In the repository's :file:`README`, we recommend that you include this section:

.. code-block:: text

   Git LFS
   -------

   To clone and use this repository, you'll need Git Large File Storage (LFS).

   Our [Developer Guide](https://developer.lsst.io/tools/git_lfs.html)
   explains how to set up Git LFS for LSST development.

.. _Homebrew: http://brew.sh
