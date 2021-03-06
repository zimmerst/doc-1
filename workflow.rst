##################################################
Development Workflow with Git, GitHub, and Jenkins
##################################################

This page describes procedures for collaborating on Fermi software.

1. :ref:`Git and repoman for development <git-setup>`.
2. :ref:`The fermi-lat GitHub organization <github-org>`.
3. :ref:`Using Git branches <git-branching>`.
4. :ref:`Code reviews <review-preparation>`.
5. :ref:`Merging code <workflow-code-review>`.
6. :ref:`Builds with Jenkins <ci-builds>`.
7. :ref:`Making a release <release-workflow>`.

.. _git-setup:

Git and repoman setup
=====================

It's recommended you install Git version 1.9.5 or later.

Follow these steps to configure your Git environment for DM work:

1. :ref:`Set Git and GitHub to use your institution-hosted email address <git-setup-institutional-email>`.
2. :ref:`Set Git to use 'plain' pushes <git-setup-plain-pushes>`.

Repoman
-------

`repoman <https://fermi-lat.github.io/repoman>`__ is a tool 
to coordinate checking out multiple branches across multiple
repositories, as well as checking out the exact state of a given
container tag, for example. It's the main way you will perform a
checkout of everything needed to build ScienceTools or
GlastRelease. Please consult the repoman documentation for more
information on how to use it.

In general, you will use repoman when you need to check everything out
for compiling, especially when you need to checkout branches in
multiple repositories. The CI jobs implemented in Jenkins also use
repoman to perform a checkout whenever a Pull Request is created or
modified.

Some examples of checking out projects:

.. code-block:: bash

   # Checks out ScienceTools at master, reads packageList.txt verbatim
   # If the packageList.txt has a ref that's incorrect, checkout should fail
   repoman checkout ScienceTools

   # Checks out ScienceTools at master, reads packageList.txt verbatim
   # refs in packages named extlist override those in packageList.txt
   repoman checkout ScienceTools extlist 

   # Checks out ScienceTools at ScienceTools-11-08-00
   # Reads packageList.txt verbatim at ScienceTools-11-08-00, refs named ScienceTools-11-08-00 override
   # If the packageList.txt has a ref that's incorrect, checkout should fail,
   # unless that package does have a tag for ScienceTools-11-08-00
   repoman checkout ScienceTools ScienceTools-11-08-00  

   # ScienceTools has branch repoman_scons_reorg, it takes precedence over master.
   # Dependencies checked out according to their development branch (e.g. master)
   # If SconsShared has tag SConsShared-02-03-00 and branch extlist
   # SConsShared-02-03-00 takes precedence, so extlist is never checked out
   repoman checkout ScienceTools --develop --force SConsShared-02-03-00 extlist repoman_scons_reorg

By default, repoman will fail if there are changes in your working
copy (that way it doesn't accidentally blow out work you've maybe done
developing and haven't committed). You will likely want to stash or
reset your repo and then try the checkout again or perform a ``--force``.


.. _github-org:

fermi-lat GitHub Organization
=============================

Fermi's Git repositories are available from the main GitHub organization,
`fermi-lat <https://github.com/fermi-lat>`__.

You should already be a member of the `fermi-lat <https://github.com/fermi-lat>`__ GitHub organization.
If you cannot create repositories or push to repositories there, you need to be added to the organization. You may contact <who?> for access.


.. _git-branching:

Git Branching Policy
====================

Rather than forking GitHub repositories, developers use a *shared repository model* by cloning repositories in the `fermi-lat <https://github.com/fermi-lat>`_ GitHub organization.
Keep in mind that this means the GitHub ``origin`` remotes are shared, so branch names and git tags are shared.

.. _git-branch-integration:

The master branch
-----------------

``master`` is the main integration branch for our repositories.
The master branch should always be stable and releaseable.
In some circumstances, a ``release`` integration branch may be used. For example, we may want to have L1 branches for the L1 version of GlastRelease.
Development is not done directly on the ``master`` branch, but instead on branches. Branches usually should go through a pull request before merging.

Documentation edits and additions are the only scenarios where working directly on ``master`` and by-passing the code review process is permitted.
In most cases, documentation writing benefits from peer editing (code review) and *can* be done on a ticket branch.

The Git history of ``master`` **must never be re-written** with force pushes.
We don't have branch protection on, but this may change, preventing
force pushes.

.. _git-branch-user:

User branches
-------------

It's sometimes convenient to do experimental, proof-of-concept work in 'user branches.'

These branches might be named as follows:

.. code-block:: text

   u/{{username}}/{{topic}}

User branches can be pushed to GitHub to enable collaboration and communication.
Before offering unsolicited code review on your colleagues' user branches, remember that the work is intended to be an early prototype.

Developers can feel free to rebase and force push work to their personal user branches.

.. _git-branch-ticket:

Ticket branches
---------------

If you are using Jira to track your work, we recommend you consider using
ticket branches.

Ticket branches are associated with a Jira ticket. SLAC's Jira is
integrate so that you may view a ticket and see any pull
requests, commits, or branches in github related to that ticket,
provided the Jira ticket ID is in the branch name (in the case of
branches, commits, and pull requests) or in the commit message, for
commits.

If the Jira ticket is named ``GRINF-NN``, then the ticket branch will be named:

.. code-block:: text

   GRINF-NN

You may also add additional text to the branch name as well. A full
branch name might look something like the following:

.. code-block:: text

   GRINF-79_st_user_support

A ticket branch can be made by branching off an existing user branch.

When code on a ticket branch is ready for review and merging, follow the :ref:`code review process documentation <workflow-code-review>`.

.. _review-preparation:

Review Preparation
==================

When development on your ticket branch is complete, we use a standard process for reviewing and merging your work.
This section describes how to prepare your work for review.

.. _workflow-pushing:

Pushing code
------------

We recommend that you organize commits, improve commit messages, and ensure that your work is made against the latest commits on ``master`` with an `interactive rebase <https://help.github.com/articles/about-git-rebase/>`_.
A common pattern is:

.. code-block:: bash

   git checkout master
   git pull
   git checkout GRINF-NN
   git rebase -i master
   # interactive rebase
   git push --force

.. _Workflow-testing:

Testing with Jenkins
--------------------

Use `Jenkins at srs.slac.stanford.edu/hudson <https://srs.slac.stanford.edu/hudson>`_ to run a build based on your branch work.
To log into Jenkins, you'll need to ask for an account from <who?>.

Jenkins finds, builds, and tests your work according to the name of your ticket branch; Stack repositories lacking your ticket branch will fall back to ``master``.

You can monitor builds in the `#jenkins <https://fermi-lat.slack.com/messages/jenkins>`_ Slack channel.
**If your build failed,** click on the **Console** link in the Slack message to see the build.

.. _workflow-pr:

Make a pull request
-------------------

On GitHub, `create a pull request <https://help.github.com/articles/creating-a-pull-request/>`_ for your branch.

The pull request's description shouldn't be exhaustive; only include information that will help frame the review.

.. _workflow-code-review:

Code Review and Merging Process
===============================

.. _workflow-review-purpose:

The scope and purpose of code review
------------------------------------

We recommend reviewing work before it is merged to ensure that code is
maintainable and usable by someone other than the author. Typically,
the package maintainer should be the reviewer. 

- Is the code well commented, structured for clarity, and consistent?
- Is there adequate unit test coverage for the code?
- Is the documentation augmented or updated to be consistent with the code changes?
- Are the Git commits well organized and well annotated to help future developers understand the code development?

Ideally you aren't reviewing your own code, but if you must, please
follow those guidelines. Discussion may be performed in Github, which
can help capture intent, but feel free to use Slack for higher
bandwidth/lower latency discussions.


.. _workflow-code-review-process:

Code review discussion
----------------------

Using GitHub pull requests
^^^^^^^^^^^^^^^^^^^^^^^^^^

Code reviews are a collaborative check-and-improve process.
Reviewers do not hold absolute authority, nor can developers ignore
the reviewer's suggestions.
The aim is to discuss, iterate, and improve the pull request until the work is ready to be deployed on ``master``.

Code review discussion should happen on the GitHub pull request, with
the reviewer giving a discussion summary and conclusive thumbs-up on
the Jira ticket.

When conducting an extensive code review in a PR, reviewers should use GitHub's `"Start a review" feature <github-review>`_.
This mode lets the reviewer queue multiple comments that are only sent once the review is submitted.
Note that GitHub allows a reviewer to classify a code review: "Comment," "Approve," or "Request changes."

.. _github-review: https://help.github.com/articles/reviewing-proposed-changes-in-a-pull-request/

Reviewers should use GitHub's `line comments`_ to discuss specific pieces of code.
As line comments are addressed, the developer may use GitHub's `emoji reactions`_ to indicate that the work is done (the "👍" works well).
Responding to each line comment isn't required, but it can help a developer track progress in addressing comments.
We discourage replies that merely say "Done" since *text* replies generate email traffic; emoji reactions aren't emailed.
Of course, use text replies if a discussion is required.

.. _line comments: https://help.github.com/articles/commenting-on-a-pull-request/#adding-line-comments-to-a-pull-request
.. _emoji reactions: https://help.github.com/articles/about-discussions-in-issues-and-pull-requests/

.. figure:: /_static/processes/workflow/reaction@2x.gif

   GitHub PR reactions are recommended for checking off completion of individual comments.

Another effective way to track progress towards addressing general review comments is with `Markdown task lists`_.

.. _Markdown task lists: https://help.github.com/articles/about-task-lists/

.. _workflow-code-review-merge:

Merging
-------

Putting a ticket in a **Reviewed** state gives the developer the go-ahead to merge the ticket branch.
If it has not been done already, the developer should rebase the ticket branch against the latest master.
During this rebase, we recommend squashing any fixup commits into the main commit implementing that feature.
Git commit history should not record the iterative improvements from code review.
If a rebase was required, a final check with Jenkins should be done.

We **always use non-fast forward merges** so that the merge point is marked in Git history, with the merge commit containing the ticket number:

.. code-block:: bash

   git checkout master
   git pull  # Sanity check; rebase ticket if master was updated.
   git merge --no-ff GRINF-NN
   git push

The ticket branch may be deleted from the GitHub remote at the time of
merge. You may want to keep it around for a bit too, use your discretion.

.. _workflow-fixing-breakage-master:

Fixing a breakage on master
^^^^^^^^^^^^^^^^^^^^^^^^^^^

In rare cases, despite the pre-merge integration testing process described :ref:`above <workflow-testing>`, a merge to master might accidentally contain an error and "break the build".
If this occurs, the merge may be reverted by anyone who notices the breakage and verifies that the merge is the cause -- unless a fix can be created, tested, reviewed, and merged very promptly.

.. _git-commit-organization-best-practices:

Appendix: Commit Organization Best Practices
============================================

.. _git-commit-organization-logical-units:

Commits should represent discrete logical changes to the code
-------------------------------------------------------------

`OpenStack has an excellent discussion of commit best practices <https://wiki.openstack.org/wiki/GitCommitMessages#Structural_split_of_changes>`_; this is recommended reading for all DM developers.
This section summarizes those recommendations.

Commits on a ticket branch should be organized into discrete, self-contained units of change.
In general, we encourage you to err on the side of more granular commits; squashing a pull request into a single commit is an anti-pattern.
A good rule-of-thumb is that if your commit *summary* message needs to contain the word 'and,' there are too many things happening in that commit.

Associating commits to a single logical change makes debugging and code audits easier:

- Git bisect is more effective for zeroing in on the change that introduced a regression.
- Git blame is more helpful for explaining why a change was made.
- Better commit organization guides reviewers through your pull request, making for more effective code reviews.
- A bad commit can more easily be reverted later with fewer side-effects.

Some edits serve only to fix white space or code style issues in existing code.
Those whitespace and style fixes should be made in separate commits from new development.
Usually it makes sense to fix whitespace and style issues in code *before* embarking on new development (or rebase those fixes to the beginning of your ticket branch).

Rebase commits from code reviews rather than having 'review feedback' commits
-----------------------------------------------------------------------------

Code review will result in additional commits that address code style, documentation and implementation issues.
Authors should rebase (i.e., ``git rebase -i master``) their ticket branch to squash the post-review fixes to the pre-review commits.
The end-goal is that a pull request, when merged, should have a coherent development story and look as if the code was written correctly the first time.

There is *no need* to retain post-review commits in order to preserve code review discussions.
So long as comments are made in the 'Conversation' and 'Files changed' tabs of the pull request GitHub will preserve that content.

.. _git-commit-message-best-practices:

Appendix: Commit Message Best Practices
=======================================

Generally you should write your commit messages in an editor, not at the prompt.
Reserve the ``git commit -m "messsage"`` pattern for 'work in progress' commits that will be rebased before code review.

We follow standard conventions for Git commit messages, which consist of a short summary line followed by body of discussion.
`Tim Pope wrote about commit message formatting <http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html>`_.

.. _git-commit-message-summary:

Writing commit summary lines
----------------------------

**Messages start with a single-line summary of 50 characters or less**.
Consider 50 characters as a hard limit; your summary will be truncated in the  GitHub UI otherwise.
Write the message in the **imperative** tense, not the past tense.
For example, "Add feature ..." and "Fix issue ..." rather than "Added feature..." and "Fixed feature...."
Ensure the summary line contains the right keywords so that someone examining can understand what parts of the codebase are being changed.
For example, it is useful to prefix the commit summary with the area of code being addressed.

.. _git-commit-message-body:

Writing commit message body content
-----------------------------------

**The message body should be wrapped at 72 character line lengths**, and contain lists or paragraphs that explain the code changes.
The commit message body describes:

- What the original issue was
- What the changes actually are and why they were made.
- What the limitations of the code are. This is especially useful for future debugging.

Git commit messages *are not* used to document the code and tell the reader how to use it.
Documentation belongs in code comments, docstrings and documentation files.

If the commit is trivial, a multi-line commit message may not be necessary.
Conversely, a long message might suggest that the :ref:`commit should be split <git-commit-organization-best-practices>`.
The code reviewer is responsible for giving feedback on the adequacy of commit messages.

The `OpenStack docs have excellent thoughts on writing great commit messages <https://wiki.openstack.org/wiki/GitCommitMessages#Information_in_commit_messages>`_.

.. _Jira: https://jira.slac.stanford.edu/
