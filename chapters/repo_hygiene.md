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
    
    Machine-specific binaries are not the only kinds of build products.
    For example, this page was composed in *markdown* and translated
    to html by the Jekyll static site generator.  The generated
    html is not in the repository on github, even though I
    generate it in my working environment.  When I push changes
    to the repository, github re-runs Jekyll to reproduce the
    html for github pages.
    
* *Configuration-specific files.*  The development environment
    usually differs from the production environment(s).  For example,
    the development environment may serve web pages on
    `http://127.0.0.1:4000` while the production environment
    serves on `https://uo-cis422.github.io`.  The URL for the server
    is configuration information that differs not only between
    development, test, and production environments, but also
    between different deployments of the same software system.
    
* *Environment-specific metadata.*  Our working environments are
    often littered with metadata that supports the tools we use.
    For example, MacOS users will find `.DS_Store` files in
    each folder.  These files contain information used by the
    MacOS finder, such as the position and appearance of
    file icons in a finder window.  If you use an interactive
    development environment like PyCharm or Visual Studio, your
    environment likely stores some additional metadata files in
    your workspace, like the `.idea` directory in the
    workspace in which I am writing these notes.
    At the very least these metadata files would cause annoying
    conflicts in the version control system, as well as being
    a distraction to new developers trying to understand a
    system from the contents of a repository.  In the worst case,
    certain kinds of metadata files (e.g., file locks) could
    trigger failures that would be difficult to debug and correct.
    
* *Foreign dependencies.*   By a *foreign* dependency,
    I mean one that is not under control of the developers.
    For example, my project may depend on a Python interpreter
    or a database interface, but they are not under my control.
    They could certainly bloat a repository, but storage is
    cheap and getting cheaper; my greater worries are not
    about machine resources.  A foreign dependency is very
    often also something that will differ, at least in configuration,
    from one environment to another.  Moreover, we often want to
    track evolution of the foreign dependency. For example,
    I might be using Jekyll version 4.0.0 at this moment, but
    I want to make it as easy as possible to upgrade to
    Jekyll 4.1.3 in a future version of this page. 
    
## Things to include

Most obviously, a repository should include all, or as
much as possible, of the source materials produced directly
by team members, including not only those that become
part of the product as seen by clients, but also anything
else needed for extension and modification:  test suites,
build scripts, design documentation, user documentation, etc. 

Some of the things that *should* be in the repository
follow from other things that *should not* be included. 

### Foreign dependencies: Recipes, not cake

Almost any substantial software project depends on
many modules that are *foreign* in the sense that they
are not built or modified by the development team.
For example, if I build a web site using Python and
the Flask web framework, I will have foreign dependencies
on a particular version of Python and on the Flask libraries.
I do not include the Python interpreter in my repo, nor
even the Flask library source code.  Instead, I provide
*recipes* for installing those dependencies.  

The simplest recipe would be a text documentation file
describing the steps required to install the foreign
dependencies.  You may have encountered such recipes: Go to
website X, download zip file Z, give the following five
shell commands, and so on.   They are tedious and
error-prone.  Some such documentation may be
unavoidable, but we want to minimize it and simplify
it as far as possible by automating any steps we can.
Fortunately various kinds of *package managers* or
*virtual environments* help with this automation. 

Package managers are more and more becoming part of the
standard infrastructure for programming languages.
Python has virtual environments and pip; Javascript has
npm; Go has cargo; Ruby has gems; etc.  A package
manager typically has an associated, very simple
scripting language or file format.  We will illustrate
with Python's virtual environments and Javascript's
npm.

### Python virtual environments

One of the jobs of a good packaging system is to
provide some *encapsulation* of a project workspace.
If I install Flask version 1.1.2 for my new project,
it should not break my older project that used
Flask 0.9.  This is the purpose of a *virtual environment*
for Python.  

A virtual environment is essentially just a directory
that holds a set of foreign dependencies for a project.
When it is "activated", an `import` statement in
a Python program will look for modules in the
virtual environment before looking elsewhere.  Each
project with its own virtual environment can have its
own set of foreign dependencies without fear of conflict
with other projects. 

But the virtual environment does not belong in the
repository!  Instead, what belongs in the repository
is a recipe for rebuilding the virtual environment.
A key part of that is a file called `requirements.txt`. 
To extract the recipe from a virtual environment, we
use `pip freeze >requirements.txt`.  To load a new
virtual environment with a matching set of foreign
dependencies, we use `pip -r requirements.txt`, which
reads the recipe from `requirements.txt` and loads
the dependencies into the environment.  The single,
simple text file `requirements.txt` is what belongs
in the repository. 

### Javascript npm  (node package manager)








