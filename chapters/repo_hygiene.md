---
layout: page
title: What goes in the repository (and what 
  should be kept out)
---

We may think of a git repository as a copy of our project
files, but it is not a copy of *all* the files of our
project.  What should be included, and what should we leave
out?  Why? 

*Why* questions almost always require us to take a step back
to consider our goals and constraints.  We'll begin there.

## The purposes of a repository

A software project repository is not just a snapshot of
a software project.  Its purposes include: 

* *Coordinating changes by team members.*
Software is built by teams, and team members can inadvertently
make changes that *conflict*. By
 itself, a
version control system like git (or subversion, or mercurial,
or ...) merely monitors the simplest kind of conflict,
preventing Mary and Tom from accidentally making different changes
to the same line of source code or other documents.
More sophisticated coordination (e.g., ensuring that
Tom is using the latest version of a function in Mary's
API) depend on team processes and norms, such as
test suites and inspection, together with the version
control system tracking and grouping sets of changes.
In git, the fundamental unit of change is a *commit*,
which may group smaller changes to several source files. 

* *Controlling deployment(s).*  Developers do not directly
modify software systems in active use, except in the
most dire emergencies.  They work in different environments
(the *development environment*), where they can make
and test many tentative changes.  Those changes must be
coordinated and tested before they enter the *production environment*,
often with an intermediate stop in a *test environment*
that mimics the production environment as closely as
possible but with fewer risks.  Because the version control
system tracks and coordinates changes, it is usual to
base *deployment* on a version in the shared repository rather than
a copy in any developer's working copy.

* *Distributing to new developers*  When a new member
joins the team, they obtain the initial version of their
development workspace from the repository.  Similarly, if
the software system is distributed to other groups of
developers who may wish to customize or extend it, they
obtain their working copies from the repository (which
they may or may not *fork* into a new repository).

From these purposes we can infer not only that certain
things must be in the repository, but also that some
other things must be kept *out* of the repository.
We will start with things that must be excluded, and
then things that must be included. 

## Things to exclude from the repository

* *Build products.*  By "build products" I mean anything
    that is produced automatically (by software, not by humans)
    while constructing the software system.  The most obvious
    examples are the outputs of compilers, e.g., `.o` files
    and executable files produced by a C compiler.  Mary uses
    Ubuntu Linux and Ted uses MacOS; while these are similar
    enough for source code compatibility, they do not use
    the same binary file formats.  Moreover, it would be annoying
    if git were constantly reporting conflicts in binary files
    that were merely side effects of changes in source files.
    
* *Configuration-specific files.*  The development environment
    usually differs from the production environment(s).  For example,
    the development environment may serve web pages on
    `http://127.0.0.1:4000` while the production environment
    serves on `https://uo-cis422.github.io`


