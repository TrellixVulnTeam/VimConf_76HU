.. _ctags(1):

==============================================================
ctags
==============================================================
--------------------------------------------------------------
Generate tag files for source code
--------------------------------------------------------------
:Version: 0.0.0
:Manual group: Universal-ctags
:Manual section: 1

SYNOPSIS
--------
|	**ctags** [options] [source_file(s)]
|	**etags** [options] [source_file(s)]


DESCRIPTION
-----------

The **ctags** and **etags** programs
(hereinafter collectively referred to as ctags,
except where distinguished) generate an index (or "tag") file for a
variety of **language objects** found in **source file(s)**. This tag file allows
these items to be quickly and easily located by a text editor or other
utilities (**client tools**). A **tag** signifies a language object for which an index entry is
available (or, alternatively, the index entry created for that object).

Alternatively, ctags can generate a cross reference
file which lists, in human readable form, information about the various
language objects found in a set of source files.

Tag index files are supported by numerous editors, which allow the user to
locate the object associated with a name appearing in a source file and
jump to the file and line which defines the name. See the manual of your
favorite editor about utilizing ctags command and
the tag index files in the editor.

ctags is capable of generating different **kinds** of tags
for each of many different **languages**. For a complete list of supported
languages, the names by which they are recognized, and the kinds of tags
which are generated for each, see the ``--list-languages`` and ``--list-kinds``
options.

This man page describes **Universal-ctags**, an implementation of ctags
derived from **Exuberant-ctags**. The major incompatible changes between
Universal-ctags and Exuberant-ctags are enumerated in
ctags-incompatibilities(7).

One of the advantages of Exuberant-ctags is that it allows a user to
define a new parser from command line. Extending this feature is one
of the major topics in Universal-ctags development. ctags-optlib(7)
describes how the feature is extended.

Newly introduced premature features are not explained here. If you
are interested in such features and ctags internal,
visit http://docs.ctags.io/en/latest/.


SOURCE FILES
------------

Unless the ``--language-force`` option is specified, the language of each source
file is automatically selected based upon a **mapping** of file names to
languages. The mappings in effect for each language may be displayed using
the ``--list-maps`` option and may be changed using the ``--langmap`` option. On
platforms which support it, if the name of a file is not mapped to a
language, ctags tries to guess the language for
the file by inspecting its content. See "Guessing parser".

All files that have no file name mapping and no guessed parser are
ignored. This permits running ctags on all files in
either a single directory (e.g.  "ctags \*"), or on
all files in an entire source directory tree
(e.g. "ctags -R"), since only those files whose
names are mapped to languages will be scanned.

The same extensions are mapped to multiple parsers. For example, ".h"
are mapped to C++, C and ObjectiveC. These mappings can cause a
trouble. ctags tries to select the proper parser
for the source file by applying kinds of heuristics to its content. However,
it is not perfect.  In that case use ``--language-force=language``,
``--langmap=map[,map[...]]``, or ``--map-<LANG>=-pattern|extension``
options. (The heuristics are applied either ``--guess-language-eagerly`` is
given or not.)

.. options should be revised here
	``--map-<LANG>`` (done)
	``--langmap=map[,map[...]]`` (done)
	``--language-force=language`` (done)
	``--languages=[+|-]list`` (done)
	``--list-maps[=language|all]`` (done)
	``--list-map-extensions`` (done)
	``--list-map-patterns`` (done)

Guessing parser
~~~~~~~~~~~~~~~

If ctags cannot select a parser from the mapping of file names,
various tests are conducted for the guessing:

template file name testing
	If the file name has ".in" extension, apply the mapping to the file
	name without the extension. For example, "config.h" is tested for a file
	named "config.h.in".

"interpreter" testing
	The first line of the file is checked to see if the file is a "#!"
	script for a recognized language.  ctags looks for
	a parser having the same name.

	If ctags finds no such parser,
	ctags looks for the name in alias lists. For
	example, consider if the first line is "#!/bin/sh".  Though
	ctags has "shell" parser, it doesn't have "sh"
	parser. However, "sh" is listed as an alias for "shell",
	ctags selects the "shell" parser for the file.

	An exception is "env". If "env" is specified, ctags
	reads more lines to find real interpreter specification.

	To display the list of aliases, use ``--list-aliases`` option.
	To add/remove an item to/from the list, use ``--alias-<LANG>=[+|-]aliasPattern``
	option.

"zsh autoload tag" testing
	If the first line is started with "#compdef" or "#autoload",
	ctags regards the line as "zsh".

"emacs mode at the first line" testing
	Emacs editor has multiple editing modes specialized to programming
	languages. Emacs can recognize a marker called modeline in a file
	and utilize the marker for the mode selection. This testing does
	the same as what Emacs does.

	ctags treats *MODE* as a name of interpreter and applies the same
	rule of "interpreter" testing if the first line has one of
	the following patterns::

		-*- mode: MODE -*-

	or

	::

		-*- MODE -*-

"emacs mode at the EOF" testing
	Emacs editor recognizes another marker at the end of file as a
	mode specifier. This testing does the same as what Emacs does.

	ctags treats *MODE* as a name of interpreter and applies the same
	rule of "interpreter" testing if the lines at the tail of the file
	have the following pattern::

		Local Variables:
		...
		mode: MODE
		...
		End:

	3000 characters are sought from the end of file to find the pattern.

"vim modeline" testing
	Like the modeline of Emacs editor, Vim editor has the same concept.
	ctags treats *TYPE* as a name of interpreter and applies the same
	rule of "interpreter" testing if the last 5 lines of the file
	have one of the following patterns::

		filetype=TYPE

	or

	::

		ft=TYPE

"PHP marker" testing
	If the first line is started with "<?php",
	ctags regards the line as "php".

Looking into the file contents is more expensive operation than file
name matching. So ctags runs the testings in limited
conditions.  "interpreter" testing is enabled only when a file is
executable or ``--guess-language-eagerly`` (``-G`` in short) option is
given. The other testings are enabled only when ``-G`` option is
given.

``--print-language`` can be used just for printing the results of
parser selections for given files instead of making tags file.

Examples:

.. code-block:: console

	$ ctags --print-language config.h.in input.m input.unknown
	config.h.in: C++
	input.m: MatLab
	input.unknown: NONE

``NONE`` means that ctags does not select any parser for the file.

.. options should be explained and revised here
   ``--list-aliases=`` (done)
   ``--alias-<LANG>=`` (done)


TAG ENTRIES
-----------

A tag is an index for a language object. Concepts of tag and related
things in Exuberant-ctags are refined and extended in Universal-ctags.

A tag is categorized into **definition tags** or **reference tags**.
Though some exceptions are, Exuberant-ctags tags only definitions of
language objects; it tags language objects introducing new names in
source files. In addition, Universal-ctags has infrastructure for
tagging references of language objects. However, the area where
reference tags are implemented is very limited in the current version.


Fields
~~~~~~

A tag can has various information. They are called **fields**. The
essential fields are **name** of language objects, **input**,
**pattern**, and **line**. ``input:`` is the name of source file where
``name:`` is defined or referenced. ``pattern:`` can be used to search
the **name** in ``input:``. **line** is the line number where
``name:`` is defined or referenced in
``input:``.

ctags offers extension fields. See also the
descriptions of ``--list-fields`` option and ``--fields`` option.


Kinds
~~~~~~

``kind:`` is a field which represents the *kind* of language object
specified by a tag. Kinds used and defined are very different between
parsers. For example, C language defines "macro", "function",
"variable", "typedef", etc. See also the descriptions of
``--list-kinds`` option and ``--kinds-<LANG>`` option.


Extras
~~~~~~

BASICALLY, ctags tags only language objects appeared
on source files as is. In other words, a value for a ``name:`` field
should be found on the source file associated with the ``name:``.  An
extra tag (*extra*) is for tagging a language object with processed
name or for tagging not associated with a language object. Typical
extra tag is "qualified". That tags a language object with
class-qualified or scope-qualified name.

The following example demonstrates "qualified" extra tag.

.. code-block:: Java

	package Bar;
	import Baz;

	class Foo {
		// ...
	}

For the source file, ctags tags "Bar" and "Foo" by
default.  If "qualified" extra is enabled from command line, "Bar.Foo"
is also tagged though the string "Bar.Foo" is not in the source code.

See also the descriptions of ``--list-extras`` option and ``--extras``
option.

Roles
~~~~~~

*Role* is a newly introduced concept in Universal-ctags. Role is a
concept related with reference tags. Role is not implemented widely yet.

A kind represents what is the language object specified with a tag.  A
role is introduced for representing how the language object specified
with a tag is referenced. Roles are defined in a kind.

For the source file used for demonstrating in "Extras" subsection,
"Baz" is tagged as a reference tag with kind "package" and with
role "imported".

See also the descriptions of ``--list-roles`` option.


Language own fields and extras
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Exuberant-ctags has concepts of fields and extras. They are common
between parsers. In addition, Universal-ctags provides language own
fields and extras.


.. options should be explained and revised here
   ``--list-languages`` (done)
   ``--list-kinds``     (done)
   ``--list-kinds-full``(done)
   ``--list-fields``    (done)
   ``--list-extras``    (done)
   ``--list-roles``     (done)
   ``--kinds-<LANG>=``  (done)
   ``--fields=``        (done)
   ``--fields-<LANG>``  (done)
   ``--extras=``        (done)
   ``--extras-<LANG>=`` (done)


COMMAND LINE INTERFACE
----------------------

Despite the wealth of available options, defaults are set so that
ctags is most commonly executed without any options (e.g.
"ctags \*", or "ctags -R"), which will
create a tag file in the current directory for all recognized source
files. The options described below are provided merely to allow custom
tailoring to meet special needs.

Note that spaces separating the single-letter options from their parameters
are optional.

Note also that the boolean parameters to the long form options (those
beginning with "--" and that take a "[=yes|no]" parameter) may be omitted,
in which case "=yes" is implied. (e.g. ``--sort`` is equivalent to ``--sort=yes``).
Note further that "=1", "=on", and "=true" are considered synonyms for "=yes",
and that "=0", "=off", and "=false" are considered synonyms for "=no".

Some options are either ignored or useful only when used while running in
etags mode (see -e option). Such options will be noted.

Most options may appear anywhere on the command line, affecting only those
files which follow the option. A few options, however, must appear
before the first file name and will be noted as such.

Options taking language names will accept those names in either upper or
lower case. See the ``--list-languages`` option for a complete list of the
built-in language names.


Letters and names
~~~~~~~~~~~~~~~~~

Some options take letters as parameters (e.g. ``--kinds-<LANG>`` option).
Specifying just letters help a user make a complicated command line
quickly.  However, the command line including sequence of the letters
becomes difficult to be understood.

Universal-ctags accepts names in
addition to such letters. The names and letters can be mixed in an
option parameter by surrounding each name by braces. Thus, for an
example, following three notations for ``--kinds-C`` option have
the same meaning::

	--kinds-C=+pLl
	--kinds-C=+{prototype}{label}{local}
	--kinds-C=+{prototype}L{local}

Note that braces may be meta characters in your shell. Put
single quotes in such case.

``--list-...`` options shows letters and associated names.


List options
~~~~~~~~~~~~

Universal-ctags introduces many ``--list-...`` options that provide
the internal data of Universal-ctags. Both users and client tools may
use the data. ``--with-list-header`` and ``--machinable`` options
adjust the output of the most of ``--list-...`` options.

The default setting (``--with-list-header=yes`` and ``--machinable=no``)
is for using interactively from a terminal. The header that explains
the meaning of columns is simply added to the output, and each column is
aligned in all lines. The header line starts with a hash ('#') character.

For scripting in a client tool, ``--with-list-header=no`` and
``--machinable=yes`` may be useful. The header is not added to the
output, and each column is separated by tab characters.

Note the order of columns will change in the future release.
However, labels in the header will not change. So by scanning
the header, a client tool can find the index for the target
column.

.. options that should be explained and revised here
   ``--list-features``    (done)
   ``--machinable``       (done)
   ``--with-list-header`` (done)

OPTION ITEMS
------------
ctags has more options listed here.
Options started from underscore character like ``--_echo=msg``
are not listed here. They are experimental or debugging purpose.

``-a``
	Equivalent to ``--append``.

``-B``
	Use backward searching patterns (e.g. ?pattern?). [Ignored in etags mode]

``-e``
	Enable etags mode, which will create a tag file for use with the Emacs
	editor. Alternatively, if ctags is invoked by a
	name containing the string "etags" (either by renaming,
	or creating a link to, the executable), etags mode will be enabled.
	This option must appear before the first file name.

``-f tagfile``
	Use the name specified by tagfile for the tag file (default is "tags",
	or "TAGS" when running in etags mode). If tagfile is specified as "-",
	then the tag file is written to standard output instead. ctags
	will stubbornly refuse to take orders if tagfile exists and
	its first line contains something other than a valid tags line. This
	will save your neck if you mistakenly type "ctags -f
	\*.c", which would otherwise overwrite your first C file with the tags
	generated by the rest! It will also refuse to accept a multi-character
	file name which begins with a '-' (dash) character, since this most
	likely means that you left out the tag file name and this option tried to
	grab the next option as the file name. If you really want to name your
	output tag file "-ugly", specify it as "./-ugly". This option must
	appear before the first file name. If this option is specified more
	than once, only the last will apply.

``-F``
	Use forward searching patterns (e.g. /pattern/) (default). [Ignored
	in etags mode]

``-G``
	Equivalent to ``--guess-language-eagerly``.

``-h list``
	Specifies a list of file extensions, separated by periods, which are
	to be interpreted as include (or header) files. To indicate files having
	no extension, use a period not followed by a non-period character
	(e.g. ".", "..x", ".x."). This option only affects how the scoping of a
	particular kinds of tags is interpreted (i.e. whether or not they are
	considered as globally visible or visible only within the file in which
	they are defined); it does not map the extension to any particular
	language. Any tag which is located in a non-include file and cannot be
	seen (e.g. linked to) from another file is considered to have file-limited
	(e.g. static) scope. No kind of tag appearing in an include file
	will be considered to have file-limited scope. If the first character
	in the list is a plus sign, then the extensions in the list will be
	appended to the current list; otherwise, the list will replace the
	current list. See, also, the ``--file-scope`` option. The default list is
	".h.H.hh.hpp.hxx.h++.inc.def". To restore the default list, specify ``-h``
	default. Note that if an extension supplied to this option is not
	already mapped to a particular language (see "SOURCE FILES", above),
	you will also need to use either the ``--langmap`` or ``--language-force`` option.

``-I identifier-list``
	Specifies a list of identifiers which are to be specially handled while
	parsing C and C++ source files. This option is specifically provided
	to handle special cases arising through the use of preprocessor macros.
	When the identifiers listed are simple identifiers, these identifiers
	will be ignored during parsing of the source files. If an identifier is
	suffixed with a '+' character, ctags will also
	ignore any parenthesis-enclosed argument list which may immediately
	follow the identifier in the source files. If two identifiers are
	separated with the '=' character, the first identifiers is replaced by
	the second identifiers for parsing purposes. The list of identifiers may
	be supplied directly on the command line or read in from a separate file.
	If the first character of identifier-list is '@', '.' or a pathname
	separator ('/' or '\'), or the first two characters specify a drive
	letter (e.g. "C:"), the parameter identifier-list will be interpreted as
	a filename from which to read a list of identifiers, one per input line.
	Otherwise, identifier-list is a list of identifiers (or identifier
	pairs) to be specially handled, each delimited by either a comma or
	by white space (in which case the list should be quoted to keep the
	entire list as one command line argument). Multiple ``-I`` options may be
	supplied. To clear the list of ignore identifiers, supply a single
	dash ("-") for identifier-list.

	This feature is useful when preprocessor macros are used in such a way
	that they cause syntactic confusion due to their presence. Indeed,
	this is the best way of working around a number of problems caused by
	the presence of syntax-busting macros in source files (see "CAVEATS").
	Some examples will illustrate this point.

	.. code-block:: C

		int foo ARGDECL4(void *, ptr, long int, nbytes)

	In the above example, the macro "ARGDECL4" would be mistakenly
	interpreted to be the name of the function instead of the correct name
	of "foo". Specifying "-I ARGDECL4" results in the correct behavior.

	.. code-block:: C

		/* creates an RCS version string in module */
		MODULE_VERSION("$Revision$")

	In the above example the macro invocation looks too much like a function
	definition because it is not followed by a semicolon (indeed, it
	could even be followed by a global variable definition that would look
	much like a K&R style function parameter declaration). In fact, this
	seeming function definition could possibly even cause the rest of the
	file to be skipped over while trying to complete the definition.
	Specifying "-I MODULE_VERSION+" would avoid such a problem.

	.. code-block:: C

		CLASS Example {
			// your content here
		};

	The example above uses "CLASS" as a preprocessor macro which expands to
	something different for each platform. For instance CLASS may be
	defined as "class __declspec(dllexport)" on Win32 platforms and simply
	"class" on UNIX. Normally, the absence of the C++ keyword "class"
	would cause the source file to be incorrectly parsed. Correct behavior
	can be restored by specifying "-I CLASS=class".

``-L file``
	Read from file a list of file names for which tags should be generated.
	If file is specified as "-", then file names are read from standard
	input. File names read using this option are processed following file
	names appearing on the command line. Options are also accepted in this
	input. If this option is specified more than once, only the last will
	apply. Note: file is read in line-oriented mode, where a new line is
	the only delimiter and non-trailing white space is considered significant,
	in order that file names containing spaces may be supplied
	(however, trailing white space is stripped from lines); this can affect
	how options are parsed if included in the input.

``-n``
	Equivalent to ``--excmd=number``.

``-N``
	Equivalent to ``--excmd=pattern``.

``-o tagfile``
	Equivalent to ``-f tagfile``.

``-R``
	Equivalent to ``--recurse``.

``-u``
	Equivalent to ``--sort=no`` (i.e. "unsorted").

``-V``
	Equivalent to ``--verbose``.

``-w``
	This option is silently ignored for backward-compatibility with the
	ctags of SVR4 Unix.

``-x``
	Print a tabular, human-readable cross reference (xref) file to standard
	output instead of generating a tag file. The information contained in
	the output includes: the tag name; the kind of tag; the line number,
	file name, and source line (with extra white space condensed) of the
	file which defines the tag. No tag file is written and all options
	affecting tag file output will be ignored. Example applications for this
	feature are generating a listing of all functions located in a source
	file (e.g. "ctags -x --c-kinds=f file"), or generating
	a list of all externally visible global variables located in a source
	file (e.g. "ctags -x --c-kinds=v --file-scope=no file").
	This option must appear before the first file name.

``--alias-<LANG>=[+|-]aliasPattern``
	Adds ('+') or removes ('-') an alias pattern to a language specified
	with *<LANG>*. ctags refers the alias pattern in
	"Guessing parser" stage.

	The parameter aliasPattern is not a list. Use this option multiple
	times in a command line to add or remove multiple alias
	patterns.

	To restore the default language aliases, specify "default" as the
	parameter aliasPattern. Using "all" for *<LANG>* has meaning in
	following two cases:

	"--alias-all="
		This clears aliases setting of all languages.

	"--alias-all=default"
		This restores the default languages aliases for all languages.

``--append[=yes|no]``
	Indicates whether tags generated from the specified files should be
	appended to those already present in the tag file or should replace them.
	This option is off by default. This option must appear before the
	first file name.

``--etags-include=file``
	Include a reference to file in the tag file. This option may be specified
	as many times as desired. This supports Emacs' capability to use a
	tag file which "includes" other tag files. [Available only in etags mode]

``--exclude=[pattern]``
	Add pattern to a list of excluded files and directories. This option may
	be specified as many times as desired. For each file name considered
	by ctags, each pattern specified using this option
	will be compared against both the complete path (e.g.
	some/path/base.ext) and the base name (e.g. base.ext) of the file, thus
	allowing patterns which match a given file name irrespective of its
	path, or match only a specific path. If appropriate support is available
	from the runtime library of your C compiler, then pattern may
	contain the usual shell wildcards (not regular expressions) common on
	Unix (be sure to quote the option parameter to protect the wildcards from
	being expanded by the shell before being passed to ctags;
	also be aware that wildcards can match the slash character, '/').
	You can determine if shell wildcards are available on your platform by
	examining the output of the ``--list-features`` option, which will include
	"wildcards" in the compiled feature list; otherwise, pattern is matched
	against file names using a simple textual comparison.

	If pattern begins with the character '@', then the rest of the string
	is interpreted as a file name from which to read exclusion patterns,
	one per line. If pattern is empty, the list of excluded patterns is
	cleared. Note that at program startup, the default exclude list contains
	"EIFGEN", "SCCS", "RCS", and "CVS", which are names of directories for
	which it is generally not desirable to descend while processing the
	``--recurse`` option.

``--excmd=type``
	Determines the type of EX command used to locate tags in the source
	file. [Ignored in etags mode]

	The valid values for type (either the entire word or the first letter
	is accepted) are:

	number
		Use only line numbers in the tag file for locating tags. This has
		four advantages:

		1.	Significantly reduces the size of the resulting tag file.
		2.	Eliminates failures to find tags because the line defining the
			tag has changed, causing the pattern match to fail (note that
			some editors, such as vim, are able to recover in many such
			instances).
		3.	Eliminates finding identical matching, but incorrect, source
			lines (see "BUGS").
		4.	Retains separate entries in the tag file for lines which are
			identical in content. In pattern mode, duplicate entries are
			dropped because the search patterns they generate are identical,
			making the duplicate entries useless.

		However, this option has one significant drawback: changes to the
		source files can cause the line numbers recorded in the tag file
		to no longer correspond to the lines in the source file, causing
		jumps to some tags to miss the target definition by one or more
		lines. Basically, this option is best used when the source code
		to which it is applied is not subject to change. Selecting this
		option type causes the following options to be ignored: ``-BF``.

	pattern
		Use only search patterns for all tags, rather than the line numbers
		usually used for macro definitions. This has the advantage of
		not referencing obsolete line numbers when lines have been added or
		removed since the tag file was generated.

	mixed
		In this mode, patterns are generally used with a few exceptions.
		For C, line numbers are used for macro definition tags. This was
		the default format generated by the original ctags and is, therefore,
		retained as the default for this option. For Fortran, line numbers
		are used for common blocks because their corresponding source lines
		are generally identical, making pattern searches useless
		for finding all matches.

``--extra=[+|-]flags|*``
	Equivalent to ``--extras=[+|-]flags|*``, which is introduced to
	make the option naming convention align to the other options
	like ``--kinds-<LANG>=`` and ``--fields=``.

	This option is kept for backward-compatibility with Exuberant-ctags.

``--extras-<LANG>=[+|-]flags|*``
	Specifies whether to include extra tag entries for certain kinds of
	information about <LANG>. Universal-ctags
	introduces language own extras. (See "Language own fields and
	extras" about the concept). This option is for controlling them.

	Specifies "all" as <LANG> to apply the parameter flags to all
	extras; all extras are enabled with specifying '*' as the
	parameter flags. If specifying nothing as the parameter flags
	("--extras-all="), all extras are disabled. These two combinations
	are useful for testing.

	Inquire the output of ``--list-extras=<LANG>`` option for the
	extras of <LANG>.

``--extras=[+|-]flags|*``
	Specifies whether to include extra tag entries for certain kinds of
	information. See also "Extras" subsection to know what are kinds.

	The parameter flags is a set of one-letter flags, each
	representing one kind of extra tag entry to include in the tag file.
	If flags is preceded by either the '+' or '-' character, the effect of
	each flag is added to, or removed from, those currently enabled;
	otherwise the flags replace any current settings. All entries are
	included  if '*' is given.

	This ``--extras=`` option is for controlling extras common in
	languages (or language in-depends extras).  Universal-ctags
	introduces language own extras. (See "Language own fields and
	extras" about the concept). Use ``--extras-<LANG>=`` option for
	controlling them.

	The meaning of major extras is as follows (one-letter flag/name):

	F/fileScope
		Equivalent to ``--file-scope``.

	f/inputFile
		Include an entry for the base file name of every source file
		(e.g. "example.c"), which addresses the first line of the file.
		If ``end:`` field is enabled, the end line number of the file
		can be attached to the tag.

	p/pseudo
		Include pseudo tags. Enabled by default unless the tag file is
		written to standard output.

	q/qualified
		Include an extra class-qualified or namespace-qualified tag entry
		for each tag which is a member of a class or a namespace.

		This may allow easier location of a specific tags when
		multiple occurrences of a tag name occur in the tag file.
		Note, however, that this could potentially more than double
		the size of the tag file.

		The actual form of the qualified tag depends upon the language
		from which the tag was derived (using a form that is most
		natural for how qualified calls are specified in the
		language). For C++ and Perl, it is in the form
		"class::member"; for Eiffel and Java, it is in the form
		"class.member".

		Note: Using backslash characters as separators forming
		qualified name in PHP. However, in tags output of
		Universal-ctags, a backslash character in a name is escaped
		with a backslash character.

		.. TODO: Write about the detail of escaping in somewhere.

	r/reference
		Include reference tags. See "TAG ENTRIES" about reference tags.

	Inquire the output of ``--list-extras`` option for the other minor
	extras.

	A name associated with an extra can be used as alternative to a
	one-letter flag. Some minor extras have no one-letters flag. In
	that case, names must be specified anyway. See "Letters and names"
	for more details.

``--fields-<LANG>=[+|-]flags|*``
	Specifies the language own fields which are to be included in
	the entries of the tag file. Universal-ctags
	introduces language own fields. (See "Language own fields and
	extras" about the concept). This option is for controlling them.

	Specifies "all" as <LANG> to apply the parameter flags to all
	fields; all fields are enabled with specifying '*' as the
	parameter flags. If specifying nothing as the parameter flags
	("--fields-all="), all extras are disabled. These two combinations
	are useful for testing.


``--fields=[+|-]flags|*``
	Specifies the available extension fields which are to be included in
	the entries of the tag file (see "TAG FILE FORMAT", below, and, "Fields", above, for more
	information).

	The parameter flags is a set of one-letter flags,
	each representing one type of extension field to include.
	Each letter or group of letters may be preceded by either '+' to add it
	to the default set, or '-' to exclude it. In the absence of any
	preceding '+' or '-' sign, only those fields explicitly listed in flags
	will be included in the output (i.e. overriding the default set). All
	fields are included if '*' is given. This option is ignored if the
	option ``--format=1`` has been specified.

	This ``--fields=`` option is for controlling fields common in
	languages (or language in-depends fields).  Universal-ctags
	introduces language own fields. (See "Language own fields and
	extras" about the concept). Use ``--fields-<LANG>=`` option for
	controlling them.


	The meaning of major fields is as follows (one-letter flag/name):

	a/access
		Access (or export) of class members

	e/end
		End lines of various items

	f/file
		File-restricted scoping. Enabled by default.

	i/inherits
		Inheritance information.

	k
		Kind of tag as a single letter. Enabled by default.
		Exceptionally this has no name.

	K
		Kind of tag as full name
		Exceptionally this has no name.

	l/language
		Language of source file containing tag

	m/implementation
		Implementation information

	n/line
		Line number of tag definition

	p/scopeKind
		Kind of scope as full name

	r/roles
		Roles assigned to the tag.
		For a definition tag, this field takes "def" as a value.

	s
		Scope of tag definition. Enabled by default.
		Exceptionally this has no name.

	S/signature
		Signature of routine (e.g. prototype or parameter list)

	t/typeref
		Type and name of a variable or typedef as "typeref:" field.
		Enabled by default.

	z/kind
		Include the "kind:" key in kind field

	Z
		Include the "scope:" key in scope field.
		Exceptionally this has no name.

	Inquire the output of ``--list-fields`` option for the other minor
	fields.

	A name associated with a field can be used as alternative to a
	one-letter flag. Some minor fields have no one-letters flag. In
	that case, names must be specified anyway. See "Letters and names"
	for more details.

``--file-scope[=yes|no]``
	Indicates whether tags scoped only for a single file (i.e. tags which
	cannot be seen outside of the file in which they are defined, such as
	"static" tags) should be included in the output. See, also, the ``-h``
	option. This option is enabled by default.

	Universal-ctags provides alternative way to control this option,
	"F/fileScope" extra, and recommends users to use the
	extra. However, this extra can cause a trouble.
	See ctags-incompatibilities(7).

``--filter[=yes|no]``
	Causes ctags to behave as a filter, reading source
	file names from standard input and printing their tags to standard
	output on a file-by-file basis. If ``--sort`` is enabled, tags are sorted
	only within the source file in which they are defined. File names are
	read from standard input in line-oriented input mode (see note for ``-L``
	option) and only after file names listed on the command line or from
	any file supplied using the ``-L`` option. When this option is enabled,
	the options ``-f``, ``-o``, and ``--totals`` are ignored. This option is quite
	esoteric and is disabled by default. This option must appear before
	the first file name.

``--filter-terminator=string``
	Specifies a string to print to standard output following the tags for
	each file name parsed when the ``--filter`` option is enabled. This may
	permit an application reading the output of ctags
	to determine when the output for each file is finished. Note that if the
	file name read is a directory and ``--recurse`` is enabled, this string will
	be printed only once at the end of all tags found for by descending
	the directory. This string will always be separated from the last tag
	line for the file by its terminating newline. This option is quite
	esoteric and is empty by default. This option must appear before
	the first file name.

``--format=level``
	Change the format of the output tag file. Currently the only valid
	values for level are 1 or 2. Level 1 specifies the original tag file
	format and level 2 specifies a new extended format containing extension
	fields (but in a manner which retains backward-compatibility with
	original vi(1) implementations). The default level is 2. This option
	must appear before the first file name. [Ignored in etags mode]

``--guess-language-eagerly``
	Looks into the file contents for guessing the proper parser.
	See "Guessing parser".

``--help``
	Prints to standard output a detailed usage description, and then exits.

``--if0[=yes|no]``
	Indicates a preference as to whether code within an "#if 0" branch of a
	preprocessor conditional should be examined for non-macro tags (macro
	tags are always included). Because the intent of this construct is to
	disable code, the default value of this option is no. Note that this
	indicates a preference only and does not guarantee skipping code within
	an "#if 0" branch, since the fall-back algorithm used to generate
	tags when preprocessor conditionals are too complex follows all branches
	of a conditional. This option is disabled by default.

``--kinddef-<LANG>=letter,name,description``
	See ctags-optlib(7).
	Be not confused this with ``--kinds-<LANG>``.

``--kinds-<LANG>=[+|-]kinds|*``
	Specifies a list of language-specific kinds of tags (or kinds) to
	include in the output file for a particular language, where <LANG> is
	case-insensitive and is one of the built-in language names (see the
	``--list-languages`` option for a complete list). The parameter kinds is a group
	of one-letter flags designating kinds of tags (particular to the language)
	to either include or exclude from the output. The specific sets of
	flags recognized for each language, their meanings and defaults may be
	list using the ``--list-kinds`` option. Each letter or group of letters
	may be preceded by either '+' to add it to, or '-' to remove it from,
	the default set. In the absence of any preceding '+' or '-' sign, only
	those kinds explicitly listed in kinds will be included in the output
	(i.e. overriding the default for the specified language).

	Specifies '*' as the parameter kinds to include all kinds implemented
	in <LANG> in the output. Further more if "all" is given as <LANG>,
	specification of the parameter kinds affects all languages defined
	in ctags. Giving "all" makes sense only when '*' is
	given as the parameter kinds.

	As an example for the C language, in order to add prototypes and
	external variable declarations to the default set of tag kinds,
	but exclude macros, use "--c-kinds=+px-d"; to include only tags for
	functions, use "--c-kinds=f".

	A name associated with a kind can be used as alternative to a
	one-letter flag. See "Letters and names" for more details.

``--<LANG>-kinds=[+|-]kinds|*``
	Equivalent to ``--kinds-<LANG>=...``. This option is kept for
	backward-compatibility with Exuberant-ctags.

``--langdef=name``
	See ctags-optlib(7).

``--langmap=map[,map[...]]``
	Controls how file names are mapped to languages (see the ``--list-maps``
	option). Each comma-separated *map* consists of the language name (either
	a built-in or user-defined language), a colon, and a list of **file
	extensions** and/or **file name patterns**. A file extension is specified by
	preceding the extension with a period (e.g. ".c"). A file name pattern
	is specified by enclosing the pattern in parentheses (e.g.
	"([Mm]akefile)").

	If appropriate support is available from the runtime
	library of your C compiler, then the file name pattern may contain the usual
	shell wildcards common on Unix (be sure to quote the option parameter to
	protect the wildcards from being expanded by the shell before being
	passed to ctags). You can determine if shell wildcards
	are available on your platform by examining the output of the
	``--list-features`` option, which will include "wildcards" in the compiled
	feature list; otherwise, the file name patterns are matched against
	file names using a simple textual comparison.

	When mapping a file extension with ``--langmap`` option,
	it will first be unmapped from any other languages. (``--map-<LANG>``
	option provides more fine-grained control.)

	If the first character in a map is a plus sign ('+'), then the extensions and
	file name patterns in that map will be appended to the current map
	for that language; otherwise, the map will replace the current map.
	For example, to specify that only files with extensions of .c and .x are
	to be treated as C language files, use "--langmap=c:.c.x"; to also add
	files with extensions of .j as Java language files, specify
	"--langmap=c:.c.x,java:+.j". To map makefiles (e.g. files named either
	"Makefile", "makefile", or having the extension ".mak") to a language
	called "make", specify "--langmap=make:([Mm]akefile).mak". To map files
	having no extension, specify a period not followed by a non-period
	character (e.g. ".", "..x", ".x.").

	To clear the mapping for a
	particular language (thus inhibiting automatic generation of tags for
	that language), specify an empty extension list (e.g. "--langmap=fortran:").
	To restore the default language mappings for a particular language,
	supply the keyword "default" for the mapping. To specify restore the
	default language mappings for all languages, specify "--langmap=default".

	Note that file name patterns are tested before file extensions when inferring
	the language of a file. This order of Universal-ctags is different from
	Exuberant-ctags. See ctags-incompatibilities(7) for the background of
	this incompatible change.

``--language-force=language``
	By default, ctags automatically selects the language
	of a source file, ignoring those files whose language cannot be
	determined (see "SOURCE FILES", above). This option forces the specified
	*language* (case-insensitive; either built-in or user-defined) to be used
	for every supplied file instead of automatically selecting the language
	based upon its extension. In addition, the special value "auto" indicates
	that the language should be automatically selected (which effectively
	disables this option).

``--languages=[+|-]list``
	Specifies the languages for which tag generation is enabled, with *list*
	containing a comma-separated list of language names (case-insensitive;
	either built-in or user-defined). If the first language of *list* is not
	preceded by either a '+' or '-', the current list (the current settings
	of enabled/disabled languages managed in ctags internally)
	will be cleared before adding or removing the languages in *list*. Until a '-' is
	encountered, each language in the *list* will be added to the current list.
	As either the '+' or '-' is encountered in the *list*, the languages
	following it are added or removed from the current list, respectively.
	Thus, it becomes simple to replace the current list with a new one, or
	to add or remove languages from the current list.

	The actual list of
	files for which tags will be generated depends upon the language
	extension mapping in effect (see the ``--langmap`` option). Note that the most of all
	languages, including user-defined languages are enabled unless explicitly
	disabled using this option. Language names included in list may be any
	built-in language or one previously defined with ``--langdef``. The default
	is "all", which is also accepted as a valid argument. See the
	``--list-languages`` option for a list of the all (built-in and user-defined)
	language names.

	Note ``--languages=`` option works cumulative way; the option can be
	specified with different arguments multiple times in a command line.

``--license``
	Prints a summary of the software license to standard output, and then exits.

``--line-directives[=yes|no]``
	Specifies whether "#line" directives should be recognized. These are
	present in the output of preprocessors and contain the line number, and
	possibly the file name, of the original source file(s) from which the
	preprocessor output file was generated. When enabled, this option will
	cause ctags to generate tag entries marked with the
	file names and line numbers of their locations original source file(s),
	instead of their actual locations in the preprocessor output. The actual
	file names placed into the tag file will have the same leading path
	components as the preprocessor output file, since it is assumed that
	the original source files are located relative to the preprocessor
	output file (unless, of course, the #line directive specifies an
	absolute path). This option is off by default. Note: This option is generally
	only useful when used together with the ``--excmd=number`` (``-n``) option.
	Also, you may have to use either the ``--langmap`` or ``--language-force`` option
	if the extension of the preprocessor output file is not known to
	ctags.

``--links[=yes|no]``
	Indicates whether symbolic links (if supported) should be followed.
	When disabled, symbolic links are ignored. This option is on by default.

``--list-aliases[=language|all]``
	Lists the aliases for either the specified language or all
	languages, and then exits. The aliases are used in when guessing
	a parser for a source file.

``--list-extras[=languages|all]``
	Lists the extras recognized for either the specified language or
	"all" languages. See "Extras" subsection to know what are extras.

	An extra can be enabled or disabled with ``--extras=`` for common
	extras in all languages, or ``--extras-<LANG>=`` for the specified
	language.  These option takes one-letter flag or name as a parameter
	for specifying an extra.

	The meaning of columns are as follows:

	LETTER
		One-letter flag. '-' means the extra does not have one-letter flag.

	NAME
		The name of extra. The name is used in ``extras:`` field.

	ENABLED
		Whether the extra is enabled or not. It takes "yes" or "no".

	LANGUAGE
		The name of language if the extra is owned by a parser.
		"NONE" means the extra is common in parsers.

	DESCRIPTION
		Human readable description for the extra.

``--list-features``
	Lists the compiled features.

``--list-fields[=language|all]``
	Lists the fields recognized for either the specified language or
	"all" languages. See "Fields" subsection to know what are fields.

	.. TODO? xref output

	A field can be enabled or disabled with ``--fields=`` for common
	extras in all languages, or ``--fields-<LANG>=`` for the specified
	language.  These option takes one-letter flag or name as a parameter
	for specifying a field.

	The meaning of columns are as follows:

	LETTER
		One-letter flag. '-' means the field does not have one-letter flag.

	NAME
		The name of field.

	ENABLED
		Whether the field is enabled or not. It takes "yes" or "no".

	LANGUAGE
		The name of language if the field is owned by a parser.
		"NONE" means the extra is common in parsers.

	JSTYPE
		Json type used in printing the value of field when "--output-format=json"
		is specified.

		Following characters are used for representing types.

		s
			string
		i
			integer
		b
			boolean (true or false)

		The representation of this field and the output format used in
		"--output-format=json" are still experimental.

	FIXED
	   Whether this field can be disabled or not. Some fields are printed always
	   in tags output. They have "yes" as the value for this column.

	DESCRIPTION
		Human readable description for the field.

``--list-kinds[=language|all]``
	Subset of ``--list-kinds-full``. This option is kept for
	backward-compatibility with Exuberant-ctags.

	This option prints only LETTER, DESCRIPTION, and ENABLED fields
	of ``--list-kinds-full`` output. However, the presentation of
	ENABLED column is different from that of ``--list-kinds-full``
	option; "[off]" follows after description if the kind is disabled,
	and nothing follows	if enabled. The most of all kinds are enabled
	by default.

	The critical weakness of this option is that this option does not
	print the name of kind. Universal-ctags introduces
	``--list-kinds-full`` because it considers that names are
	important.

	This option does not work with ``--machinable`` nor
	``--with-list-header``.

``--list-kinds-full[=language|all]``
	Lists the tag kinds recognized for either the specified language
	or "all" languages, and then exits. See "Kinds" subsection to
	know what are kinds.

	Each kind of tag recorded in the tag file is represented by a
	one-letter flag, or name. They are also used to filter the tags
	placed into the output through use of the ``--kinds-<LANG>``
	option.

	The meaning of columns are as follows:

	LANGUAGE
		The name of language having the kind.

	LETTER
		One-letter flag. This must be unique in a language.

	NAME
		Name of the kind. This can be used as the alternative
		one-letter flag described above. If enabling 'K' field with
		``--fields=+K``, ctags uses name instead of
		letter in tags output. To enable/disable a kind with
		``--kinds-<LANG>`` option, name surrounded by braces instead
		of letter. See "Letters and names" for details. This must be
		unique in a language.

	ENABLED
		Whether the kind is enabled or not. It takes "yes" or "no".

	REFONLY
		Whether the kind is specialized for reference tagging or not.
		If the column is "yes", the kind is for reference tagging, and
		it is never used for definition tagging. See also "TAG ENTRIES".

	NROLES
		The number of roles this kind has. See also "Roles".

	MASTER
		The master parser controlling enablement of the kind.
		A kind belongs to a language (owner) in Universal-ctags;
		enabling and disabling a kind in a language has no effect on
		a kind in another language even if both kinds has the
		same letter and/or the same name. In other words,
		the namespace of kinds are separated by language.

		However, Exuberant-ctags does not separate the kinds of C and
		C++. Enabling/disabling kindX in C language enables/disables a
		kind in C++ language having the same name with kindX. To
		emulate this behavior in Universal-ctags, a concept named
		"master parser" is introduced. Enabling/disabling some kinds
		are synchronized under the control of a master language.

		.. code-block:: console

			$ ctags --kinds-C=+'{local}' --list-kinds-full \
			  | grep -E '^(#|C\+\+ .* local)'
			#LANGUAGE  LETTER NAME   ENABLED REFONLY NROLES MASTER DESCRIPTION
			C++        l      local  yes     no      0      C      local variables
			$ ctags --kinds-C=-'{local}' --list-kinds-full \
			  | grep -E '^(#|C\+\+ .* local)'
			#LANGUAGE  LETTER NAME   ENABLED REFONLY NROLES MASTER DESCRIPTION
			C++        l      local  no      no      0      C      local variables

		You see "ENABLED" field of "local" kind of C++ language is changed
		Though "local" kind of C language is enabled/disabled. If you swap the languages, you
		see the same result.

	DESCRIPTION
		Human readable description for the kind.

``--list-languages``
	Lists the names of the languages understood by ctags,
	and then exits. These language names are case insensitive and may be
	used in many other options like ``--language-force``,
	``--languages``, ``--kinds-<LANG>``, ``--regex-<LANG>``, and so on.

	Each language listed is disabled if followed by "[disabled]".
	To use the parser for such a language, specify the language as an
	argument of ``--languages=+`` option.

	This option does not work with ``--machinable`` nor
	``--with-list-header``.

``--list-map-extensions[=language|all]``
	Lists the file extensions which associate a file
	name with a language for either the specified *language* or **all**
	languages, and then exits.

``--list-map-patterns[=language|all]``
	Lists the file name patterns which associate a file
	name with a language for either the specified *language* or **all**
	languages, and then exits.

``--list-maps[=language|all]``
	Lists file name patterns and the file extensions which associate a file
	name with a language for either the specified *language* or **all**
	languages, and then exits. See the ``--langmap`` option, and "SOURCE FILES", above.

	To list the file extensions or file name patterns individually, use
	``--list-map-extensions`` or ``--list-map-patterns`` option.
	See the ``--langmap`` option, and "SOURCE FILES", above.

	This option does not work with ``--machinable`` nor
	``--with-list-header``.

``--list-regex-flags``
	See ctags-optlib(7).

``--list-roles[=language|all[.kinds]]``
	List the roles for either the specified language or "all"
	languages. If the parameter kinds is given after the parameter
	language or "all" with concatenating with '.', list only roles
	defined in the kinds. Both one-letter flags and names surrounded
	by braces are acceptable as the parameter kinds.

	The meaning of columns are as follows:

	LANGUAGE
		Name of language having the role.

	KIND(L/N)
		One-letter flag and name of kind having the role.

	NAME
		Name of the role.

	ENABLED
		Whether the kind is enabled or not. It takes "yes" or "no".
		(Currently all roles are enabled. No option for disabling
		a specified role is not implemented yet.)

	DESCRIPTION
		Human readable description for the role.

``--machinable[=yes|no]``
	Use tab character as separators for ``--list-`` option output.  It
	may be suitable for scripting. See "List options" for considered
	use cases. Disabled by default.

``--map-<LANG>=[+|-]extension|pattern``
	This option provides the way to control mapping(s) of file names to
	languages more fine-grained way than ``--langmap`` option.

	In ctags, more than one language can map to a
	file name pattern or file extension (*N:1 map*). Alternatively,
	``--langmap`` option handle only *1:1 map*, only one language
	mapping to one file name pattern or file extension.  A typical N:1
	map is seen in C++ and ObjectiveC language; both languages have
	a map to ".h" as a file extension.

	A file extension is specified by preceding the extension with a period (e.g. ".c").
	A file name pattern is specified by enclosing the pattern in parentheses (e.g.
	"([Mm]akefile)"). A prefixed plus ('+') sign is for adding, and
	minus ('-') is for removing. No prefix means replacing the map of *<LANG>*.

	Unlike ``--langmap``, *extension* (or *pattern*) is not a list.
	``--map-<LANG>`` takes one *extension* (or *pattern*). However,
	the option can be specified with different arguments multiple times
	in a command line.

``--maxdepth``
	Limits the depth of directory recursion enabled with the ``--recurse``
	(``-R``) option.

``--optlib-dir=[+]directory``
	Add an optlib *directory* to or reset **optlib** path list.
	By default, the optlib path list is empty.

``--options=pathname``
	Read additional options from file or directory.

	ctags searches *pathname* in optlib path list
	first. If ctags cannot find a file or directory
	in the list, ctags reads a file or directory
	at the specified *pathname*.

	If a file is specified, it should contain one option per line. If
	a directory is specified, files suffixed with ".ctags" under it
	are read in alphabetical order.

	As a special case, if "--options=NONE" is specified as the first
	option on the command line, preloading is disabled; the option
	will disable the automatic reading of any configuration options
	from either a file or the environment (see "FILES").

``--options-maybe=pathname``
	Same as ``--options`` but doesn't cause an error if file
	(or directory) specified with *pathname* doesn't exist.

``--print-language``
	Just prints the parsers for specified source files, and then exits.

``--quiet[=yes|no]``
	Write fewer messages (default is no).

``--recurse[=yes|no]``
	Recurse into directories encountered in the list of supplied files.
	If the list of supplied files is empty and no file list is specified with
	the -L option, then the current directory (i.e. ".") is assumed.
	Symbolic links are followed. If you don't like these behaviors, either
	explicitly specify the files or pipe the output of find(1) into
	ctags -L- instead. Note: This option is not supported on
	all platforms at present. It is available if the output of the ``--help``
	option includes this option. See, also, the ``--exclude`` to limit
	recursion.

``--regex-<LANG>=/regexp/replacement/[kind-spec/][flags]``
	See ctags-optlib(7).

``--sort[=yes|no|foldcase]``
	Indicates whether the tag file should be sorted on the tag name
	(default is yes). Note that the original vi(1) required sorted tags.
	The foldcase value specifies case insensitive (or case-folded) sorting.
	Fast binary searches of tag files sorted with case-folding will require
	special support from tools using tag files, such as that found in the
	ctags readtags library, or Vim version 6.2 or higher
	(using "set ignorecase"). This option must appear before the first file
	name. [Ignored in etags mode]

``--tag-relative[=yes|no]``
	Indicates that the file paths recorded in the tag file should be
	relative to the directory containing the tag file, rather than relative
	to the current directory, unless the files supplied on the command line
	are specified with absolute paths. This option must appear before the
	first file name. The default is yes when running in etags mode (see
	the ``-e`` option), no otherwise.

``--totals[=yes|no]``
	Prints statistics about the source files read and the tag file written
	during the current invocation of ctags. This option
	is off by default. This option must appear before the first file name.

``--undef[=yes|no]``
	Specifies whether a macro tag should be generated from an #undef CPP
	directive (in a C/C++ file), as if it were a #define directive. This
	option is enabled by default.

``--verbose[=yes|no]``
	Enable verbose mode. This prints out information on option processing
	and a brief message describing what action is being taken for each file
	considered by ctags. Normally, ctags
	does not read command line arguments until after options are read
	from the configuration files (see "FILES", below) and the CTAGS
	environment variable. However, if this option is the first argument on
	the command line, it will take effect before any options are read from
	these sources. The default is no.

``--with-list-header[=yes|no]``
	Print headers describing columns in ``--list-`` option output.
	See also "List options".

``--version``
	Prints a version identifier for ctags to standard
	output, and then exits. This is guaranteed to always contain the string
	"Universal Ctags".


OPERATIONAL DETAILS
-------------------
As ctags considers each file name in turn, it tries to
determine the language of the file by applying the following three tests
in order: if the file extension has been mapped to a language, if the
filename matches a shell pattern mapped to a language, and finally if the
file is executable and its first line specifies an interpreter using the
Unix-style "#!" specification (if supported on the platform). If a
language was identified, the file is opened and then the appropriate
language parser is called to operate on the currently open file. The parser
parses through the file and adds an entry to the tag file for each
language object it is written to handle. See "TAG FILE FORMAT", below, for
details on these entries.

This implementation of ctags imposes no formatting
requirements on C code as do legacy implementations. Older implementations
of ctags tended to rely upon certain formatting assumptions in order to
help it resolve coding dilemmas caused by preprocessor conditionals.

In general, ctags tries to be smart about conditional
preprocessor directives. If a preprocessor conditional is encountered
within a statement which defines a tag, ctags follows
only the first branch of that conditional (except in the special case of
"#if 0", in which case it follows only the last branch). The reason for
this is that failing to pursue only one branch can result in ambiguous
syntax, as in the following example:

.. code-block:: C

	#ifdef TWO_ALTERNATIVES
	struct {
	#else
	union {
	#endif
		short a;
		long b;
	}

Both branches cannot be followed, or braces become unbalanced and
ctags would be unable to make sense of the syntax.

If the application of this heuristic fails to properly parse a file,
generally due to complicated and inconsistent pairing within the
conditionals, ctags will retry the file using a
different heuristic which does not selectively follow conditional
preprocessor branches, but instead falls back to relying upon a closing
brace ("}") in column 1 as indicating the end of a block once any brace
imbalance results from following a #if conditional branch.

ctags will also try to specially handle arguments lists
enclosed in double sets of parentheses in order to accept the following
conditional construct:

	extern void foo __ARGS((int one, char two));

Any name immediately preceding the "((" will be automatically ignored and
the previous name will be used.

C++ operator definitions are specially handled. In order for consistency
with all types of operators (overloaded and conversion), the operator
name in the tag file will always be preceded by the string "operator "
(i.e. even if the actual operator definition was written as "operator<<").

After creating or appending to the tag file, it is sorted by the tag name,
removing identical tag lines.


TAG FILE FORMAT
---------------

When not running in etags mode, each entry in the tag file consists of a
separate line, each looking like this in the most general case:

tag_name<TAB>file_name<TAB>ex_cmd;"<TAB>extension_fields

The fields and separators of these lines are specified as follows:

	1.	tag name
	2.	single tab character
	3.	name of the file in which the object associated with the tag is located
	4.	single tab character
	5.	EX command used to locate the tag within the file; generally a
		search pattern (either /pattern/ or ?pattern?) or line number (see
		``--excmd``). Tag file format 2 (see ``--format``) extends this EX command
		under certain circumstances to include a set of extension fields
		(described below) embedded in an EX comment immediately appended
		to the EX command, which leaves it backward-compatible with original
		vi(1) implementations.

A few special tags are written into the tag file for internal purposes.
These tags are composed in such a way that they always sort to the top of
the file. Therefore, the first two characters of these tags are used a magic
number to detect a tag file for purposes of determining whether a
valid tag file is being overwritten rather than a source file.

Note that the name of each source file will be recorded in the tag file
exactly as it appears on the command line. Therefore, if the path you
specified on the command line was relative to the current directory, then
it will be recorded in that same manner in the tag file. See, however,
the ``--tag-relative`` option for how this behavior can be modified.

Extension fields are tab-separated key-value pairs appended to the end of
the EX command as a comment, as described above. These key value pairs
appear in the general form "key:value". Their presence in the lines of the
tag file are controlled by the ``--fields`` option. The possible keys and
the meaning of their values are as follows:

access
	Indicates the visibility of this class member, where value is specific
	to the language.

file
	Indicates that the tag has file-limited visibility. This key has no
	corresponding value.

kind
	Indicates the type, or kind, of tag. Its value is either one of the
	corresponding one-letter flags described under the various
	``--<LANG>-kinds`` options above, or a full name. It is permitted
	(and is, in fact, the default) for the key portion of this field to be
	omitted. The optional behaviors are controlled with the ``--fields`` option.

implementation
	When present, this indicates a limited implementation (abstract vs.
	concrete) of a routine or class, where value is specific to the
	language ("virtual" or "pure virtual" for C++; "abstract" for Java).

inherits
	When present, value. is a comma-separated list of classes from which
	this class is derived (i.e. inherits from).

signature
	When present, value is a language-dependent representation of the
	signature of a routine. A routine signature in its complete form
	specifies the return type of a routine and its formal argument list.
	This extension field is presently supported only for C-based
	languages and does not include the return type.

In addition, information on the scope of the tag definition may be
available, with the key portion equal to some language-dependent construct
name and its value the name declared for that construct in the program.
This scope entry indicates the scope in which the tag was found.
For example, a tag generated for a C structure member would have a scope
looking like "struct:myStruct".


HOW TO USE WITH VI
------------------

Vi will, by default, expect a tag file by the name "tags" in the current
directory. Once the tag file is built, the following commands exercise
the tag indexing feature:

vi -t tag
	Start vi and position the cursor at the file and line where "tag"
	is defined.

:ta tag
	Find a tag.

Ctrl-]
	Find the tag under the cursor.

Ctrl-T
	Return to previous location before jump to tag (not widely implemented).


HOW TO USE WITH GNU EMACS
-------------------------

Emacs will, by default, expect a tag file by the name "TAGS" in the
current directory. Once the tag file is built, the following commands
exercise the tag indexing feature:

M-x visit-tags-table <RET> FILE <RET>
	Select the tag file, "FILE", to use.

M-. [TAG] <RET>
	Find the first definition of TAG. The default tag is the identifier
	under the cursor.

M-*
	Pop back to where you previously invoked "M-.".

C-u M-.
	Find the next definition for the last tag.

For more commands, see the Tags topic in the Emacs info document.


HOW TO USE WITH NEDIT
---------------------

NEdit version 5.1 and later can handle the new extended tag file format
(see ``--format``). To make NEdit use the tag file, select "File->Load Tags
File". To jump to the definition for a tag, highlight the word, then press
Ctrl-D. NEdit 5.1 can read multiple tag files from different
directories. Setting the X resource nedit.tagFile to the name of a tag
file instructs NEdit to automatically load that tag file at startup time.


CAVEATS
-------

Because ctags is neither a preprocessor nor a compiler,
use of preprocessor macros can fool ctags into either
missing tags or improperly generating inappropriate tags. Although
ctags has been designed to handle certain common cases,
this is the single biggest cause of reported problems. In particular,
the use of preprocessor constructs which alter the textual syntax of C
can fool ctags. You can work around many such problems
by using the ``-I`` option.

Note that since ctags generates patterns for locating
tags (see the ``--excmd`` option), it is entirely possible that the wrong line
may be found by your editor if there exists another source line which is
identical to the line containing the tag. The following example
demonstrates this condition:

.. code-block:: C

	int variable;

	/* ... */
	void foo(variable)
	int variable;
	{
		/* ... */
	}

Depending upon which editor you use and where in the code you happen to be,
it is possible that the search pattern may locate the local parameter
declaration in foo() before it finds the actual global variable definition,
since the lines (and therefore their search patterns are identical).
This can be avoided by use of the ``--excmd=n`` option.

BUGS
----

ctags has more options than ls(1).

When parsing a C++ member function definition (e.g. "className::function"),
ctags cannot determine whether the scope specifier
is a class name or a namespace specifier and always lists it as a class name
in the scope portion of the extension fields. Also, if a C++ function
is defined outside of the class declaration (the usual case), the access
specification (i.e. public, protected, or private) and implementation
information (e.g. virtual, pure virtual) contained in the function
declaration are not known when the tag is generated for the function
definition. It will, however be available for prototypes (e.g. "--c++-kinds=+p").

No qualified tags are generated for language objects inherited into a class.

ENVIRONMENT VARIABLES
---------------------

CTAGS
	If this environment variable exists, it will be expected to contain a
	set of default options which are read when ctags
	starts, after the configuration files listed in FILES, below, are read,
	but before any command line options are read. Options appearing on
	the command line will override options specified in this variable.
	Only options will be read from this variable. Note that all white space
	in this variable is considered a separator, making it impossible to pass
	an option parameter containing an embedded space. If this is a problem,
	use a configuration file instead.

ETAGS
	Similar to the CTAGS variable above, this variable, if found, will be
	read when etags starts. If this variable is not
	found, etags will try to use CTAGS instead.

TMPDIR
	On Unix-like hosts where mkstemp() is available, the value of this
	variable specifies the directory in which to place temporary files.
	This can be useful if the size of a temporary file becomes too large
	to fit on the partition holding the default temporary directory
	defined at compilation time. ctags creates temporary
	files only if either (1) an emacs-style tag file is being
	generated, (2) the tag file is being sent to standard output, or
	(3) the program was compiled to use an internal sort algorithm to sort
	the tag files instead of the sort utility of the operating system.
	If the sort utility of the operating system is being used, it will
	generally observe this variable also. Note that if ctags
	is setuid, the value of TMPDIR will be ignored.

FILES
-----


$HOME/.ctags.d/\*.ctags

$HOMEDRIVE$HOMEPATH/ctags.d/\*.ctags (on MSWindows only)

.ctags.d/\*.ctags

ctags.d/\*.ctags

	If any of these configuration files exist, each will be expected to
	contain a set of default options which are read in the order listed
	when ctags starts, but before the CTAGS environment
	variable is read or any command line options are read. This makes it
	possible to set up personal or project-level defaults. It
	is possible to compile ctags to read an additional
	configuration file before any of those shown above, which will be
	indicated if the output produced by the ``--version`` option lists the
	"custom-conf" feature. Options appearing in the CTAGS environment
	variable or on the command line will override options specified in these
	files. Only options will be read from these files. Note that the option
	files are read in line-oriented mode in which spaces are significant
	(since shell quoting is not possible) but spaces at the beginning
	of a line are ignored. Each line of the file is read as
	one command line parameter (as if it were quoted with single quotes).
	Therefore, use new lines to indicate separate command-line arguments.
	A line starting with '#' is treated as a comment.

	\*.ctags files in a directory are loaded in alphabetical order.

tags
	The default tag file created by ctags.

TAGS
	The default tag file created by etags.


SEE ALSO
--------

See ctags-optlib(7) for defining (or extending) a parser
in a configuration file.

The official Universal-ctags web site at:

https://ctags.io/

Also ex(1), vi(1), elvis, or, better yet, vim, the official editor of ctags.
For more information on vim, see the VIM Pages web site at:

http://www.vim.org/


AUTHOR
------

Universal-ctags project
https://ctags.io

Darren Hiebert <dhiebert@users.sourceforge.net>
http://DarrenHiebert.com/


MOTIVATION
----------

"Think ye at all times of rendering some service to every member of the
human race."

"All effort and exertion put forth by man from the fullness of his heart is
worship, if it is prompted by the highest motives and the will to do
service to humanity."

-- From the Baha'i Writings

CREDITS
-------
This version of ctags (Universal-ctags) derived from
the repository, known as **fishman-ctags**, started by Reza Jelveh.

Some parsers are taken from **tagmanager** of **Geany** (https://www.geany.org/)
project.


The fishman-ctags was derived from Exuberant-ctags.

Exuberant-ctags was originally derived from and
inspired by the ctags program by Steve Kirkendall <kirkenda@cs.pdx.edu>
that comes with the Elvis vi clone (though virtually none of the original
code remains).

Credit is also due Bram Moolenaar <Bram@vim.org>, the author of vim,
who has devoted so much of his time and energy both to developing the editor
as a service to others, and to helping the orphans of Uganda.

The section entitled "HOW TO USE WITH GNU EMACS" was shamelessly stolen
from the info page for GNU etags.
