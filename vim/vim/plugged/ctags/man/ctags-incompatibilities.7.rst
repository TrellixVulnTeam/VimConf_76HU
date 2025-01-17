.. _ctags-incompatibilities(7):

==============================================================
ctags
==============================================================
--------------------------------------------------------------
Incompatibilities between Universal-ctags and Exuberant-ctags
--------------------------------------------------------------
:Version: 0.0.0
:Manual group: Universal-ctags
:Manual section: 7

SYNOPSIS
--------
|	**ctags** [options] [file(s)]
|	**etags** [options] [file(s)]

DESCRIPTION
-----------

This page describes major incompatible changes introduced to
Universal-ctags forked from Exuberant-ctags.

Incompatibilities in command line interface
-------------------------------------------------------------

The order of application of patterns and extensions in ``--langmap``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When applying mappings for a name of given source file,
Exuberant-ctags tests file name patterns AFTER file extensions
(**e-map-order**). Universal-ctags does this differently; it tests file
name patterns BEFORE file extensions (**u-map-order**).

This incompatible change is introduced to deal with the following
situation:

	* "build.xml" as a source file,
	* The "Ant" parser declares it handles a file name pattern "build.xml", and
	* The "XML" parser declares it handles a file extension "xml".

Which parser should be used for parsing "build.xml"?  The assumption
of Universal-ctags is the user may want to use the "Ant" parser; the
file name pattern it declares is more specific than the file extension
that the "XML" parser declares. However, e-map-order chooses the "XML"
parser.

So Universal-ctags uses the u-map-order even though it introduces an
incompatibility.

``--list-map-extensions=language`` and ``--list-map-patterns=language``
options are helpful to verify and the file extensions and the file
name patterns of given *language*.

Unexpected synchronization between ``--file-scope`` option and "F/fileScope" extra
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Universal-ctags introduces "F/fileScope" extra as the alternative to
``--file-scope`` option.

Providing the two way to control the same thing in Universal-ctags
internal can cause a trouble.

A user, expecting "--file-scope=yes" is enabled by default, gives
"--extras=q". The intention of the user may be just enabling
"q/qualified". However, "--extras=q" is evaluated as "disabling all
extras including F/fileScope, then enabling only
q/qualified". Unexpectedly the command line becomes as if
"--file-scope=no" is set.

In this case the user should set "--extras=+q" instead of "--extras=q".

Obsoleting ``--<LANG>-kinds`` option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some options have *<LANG>* as parameterized parts in their name like
``--foo-<LANG>=...`` or ``--<LANG>-foo=...``. The most of all such
options in Exuberant-ctags have the former form, ``--foo-<LANG>=...``.
The exception is ``--<LANG>-kinds``.

Universal-ctags uses the former form for all *<LANG>* parameterized
option. Use ``--kinds-<LANG>`` instead of ``--<LANG>-kinds`` in
Universal-ctags. ``--<LANG>-kinds`` still works but it will be
removed in the future.

The former form may be friendly to shell completion engines.

Disallowing to define a kind with "file" as name
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The kind name "file" is reserved.  Using it as part of kind spec in
``--regex-<LANG>`` option is now disallowed.

Incompatibilities in tags file format
-------------------------------------------------------------


Option files loading at starting up time (preload files)
-------------------------------------------------------------

File paths for preload files are changed.
Universal-ctags doesn't load "~/.ctags" at starting up time.
See "FILES" section of ctags(1).

Kind letters and names
-------------------------------------------------------------

A kind letter "F" and a kind name "file" are reserved in the
main part. A parser cannot have a kind conflicting with
these reserved ones. Some incompatible changes are introduced
to follow the above rule.

* Cobol's "file" kind is renamed to "fileDesc" because the
  kind name "file" is reserved.

* Ruby's "F" (singletonMethod) is changed to "S".

* SQL's "F" (field) is changed to "E".
