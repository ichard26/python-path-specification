
PathSpec
========

*pathspec* is a utility library for pattern matching of file paths. So
far this only includes Git's wildmatch pattern matching which itself is
derived from Rsync's wildmatch. Git uses wildmatch for its `gitignore`_
files.

.. _`gitignore`: http://git-scm.com/docs/gitignore


Tutorial
--------

Say you have a "Projects" directory and you want to back it up, but only
certain files, and ignore others depending on certain conditions::

	>>> import pathspec
	>>> # The gitignore-style patterns for files to select, but we're including
	>>> # instead of ignoring.
	>>> spec_text = """
	...
	... # This is a comment because the line begins with a hash: "#"
	...
	... # Include several project directories (and all descendants) relative to
	... # the current directory. To reference a directory you must end with a
	... # slash: "/"
	... /project-a/
	... /project-b/
	... /project-c/
	...
	... # Patterns can be negated by prefixing with exclamation mark: "!"
	...
	... # Ignore temporary files beginning or ending with "~" and ending with
	... # ".swp".
	... !~*
	... !*~
	... !*.swp
	...
	... # These are python projects so ignore compiled python files from
	... # testing.
	... !*.pyc
	...
	... # Ignore the build directories but only directly under the project
	... # directories.
	... !/*/build/
	...
	... """

We want to use the ``GitWildMatchPattern`` class to compile our patterns. The
``PathSpec`` class provides an interface around pattern implementations::

	>>> spec = pathspec.PathSpec.from_lines(pathspec.patterns.GitWildMatchPattern, spec_text.splitlines())

That may be a mouthful but it allows for additional patterns to be implemented
in the future without them having to deal with anything but matching the paths
sent to them. ``GitWildMatchPattern`` is the implementation of the actual
pattern which internally gets converted into a regular expression. ``PathSpec``
is a simple wrapper around a list of compiled patterns.

To make things simpler, we can use the registered name for a pattern class
instead of always having to provide a reference to the class itself. The
``GitWildMatchPattern`` class is registered as **gitwildmatch**::

	>>> spec = pathspec.PathSpec.from_lines('gitwildmatch', spec_text.splitlines())

If we wanted to manually compile the patterns we can just do the following::

	>>> patterns = map(pathspec.patterns.GitWildMatchPattern, spec_text.splitlines())
	>>> spec = PathSpec(patterns)

``PathSpec.from_lines()`` is simply a class method which does just that.

If you want to load the patterns from file, you can pass the file instance
directly as well::

	>>> with open('patterns.list', 'r') as fh:
	>>>     spec = pathspec.PathSpec.from_lines('gitwildmatch', fh)

You can perform matching on a whole directory tree with::

	>>> matches = spec.match_tree('path/to/directory')

Or you can perform matching on a specific set of file paths with::

	>>> matches = spec.match_files(file_paths)

Or check to see if an individual file matches::

	>>> is_matched = spec.match_file(file_path)

There is a specialized class, ``pathspec.GitIgnoreSpec``, which more closely
implements the behavior of **gitignore**. This uses ``GitWildMatchPattern``
pattern by default and handles some edge cases differently from the generic
``PathSpec`` class. ``GitIgnoreSpec`` can be used without specifying the pattern
factory::

	>>> spec = pathspec.GitIgnoreSpec.from_lines(spec_text.splitlines())


License
-------

*pathspec* is licensed under the `Mozilla Public License Version 2.0`_. See
`LICENSE`_ or the `FAQ`_ for more information.

In summary, you may use *pathspec* with any closed or open source project
without affecting the license of the larger work so long as you:

- give credit where credit is due,

- and release any custom changes made to *pathspec*.

.. _`Mozilla Public License Version 2.0`: http://www.mozilla.org/MPL/2.0
.. _`LICENSE`: LICENSE
.. _`FAQ`: http://www.mozilla.org/MPL/2.0/FAQ.html


Source
------

The source code for *pathspec* is available from the GitHub repo
`cpburnz/python-pathspec`_.

.. _`cpburnz/python-pathspec`: https://github.com/cpburnz/python-pathspec


Installation
------------

*pathspec* is available for install through `PyPI`_::

	pip install pathspec

*pathspec* can also be built from source. The following packages will be
required:

- `build`_ (>=0.6.0)
- `setuptools`_ (>=40.8.0)

*pathspec* can then be built and installed with::

	python -m build
	pip install dist/pathspec-*-py3-none-any.whl

.. _`PyPI`: http://pypi.python.org/pypi/pathspec
.. _`build`: https://pypi.org/project/build/
.. _`setuptools`: https://pypi.org/project/setuptools/


Documentation
-------------

Documentation for *pathspec* is available on `Read the Docs`_.

.. _`Read the Docs`: https://python-path-specification.readthedocs.io


Other Languages
---------------

The related project `pathspec-ruby`_ (by *highb*) provides a similar library as
a `Ruby gem`_.

.. _`pathspec-ruby`: https://github.com/highb/pathspec-ruby
.. _`Ruby gem`: https://rubygems.org/gems/pathspec



Change History
==============


0.10.1 (2022-09-02)
-------------------

Bug fixes:

- Fix documentation on `pathspec.pattern.RegexPattern.match_file()`.
- `Issue #60`_: Remove redundant wheel dep from pyproject.toml.
- `Issue #61`_: Dist failure for Fedora, CentOS, EPEL.
- `Issue #62`_: Since version 0.10.0 pure wildcard does not work in some cases.

Improvements:

- Restore support for legacy installations using `setup.py`. See `Issue #61`_.


.. _`Issue #60`: https://github.com/cpburnz/python-pathspec/pull/60
.. _`Issue #61`: https://github.com/cpburnz/python-pathspec/issues/61
.. _`Issue #62`: https://github.com/cpburnz/python-pathspec/issues/62


0.10.0 (2022-08-30)
-------------------

Major changes:

- Dropped support of EOL Python 2.7, 3.5, 3.6. See `Issue #47`_.
- The *gitwildmatch* pattern `dir/*` is now handled the same as `dir/`. This means `dir/*` will now match all descendants rather than only direct children. See `Issue #19`_.
- Added `pathspec.GitIgnoreSpec` class (see new features).
- Changed build system to `pyproject.toml`_ and build backend to `setuptools.build_meta`_ which may have unforeseen consequences.
- Renamed GitHub project from `python-path-specification`_ to `python-pathspec`_. See `Issue #35`_.

API changes:

- Deprecated: `pathspec.util.match_files()` is an old function no longer used.
- Deprecated: `pathspec.match_files()` is an old function no longer used.
- Deprecated: `pathspec.util.normalize_files()` is no longer used.
- Deprecated: `pathspec.util.iter_tree()` is an alias for `pathspec.util.iter_tree_files()`.
- Deprecated: `pathspec.iter_tree()` is an alias for `pathspec.util.iter_tree_files()`.
-	Deprecated: `pathspec.pattern.Pattern.match()` is no longer used. Use or implement
	`pathspec.pattern.Pattern.match_file()`.

New features:

- Added class `pathspec.gitignore.GitIgnoreSpec` (with alias `pathspec.GitIgnoreSpec`) to implement *gitignore* behavior not possible with standard `PathSpec` class. The particular *gitignore* behavior implemented is prioritizing patterns matching the file directly over matching an ancestor directory.

Bug fixes:

- `Issue #19`_: Files inside an ignored sub-directory are not matched.
- `Issue #41`_: Incorrectly (?) matches files inside directories that do match.
- `Issue #51`_: Refactor deprecated unittest aliases for Python 3.11 compatibility.
- `Issue #53`_: Symlink pathspec_meta.py breaks Windows.
- `Issue #54`_: test_util.py uses os.symlink which can fail on Windows.
- `Issue #55`_: Backslashes at start of pattern not handled correctly.
- `Issue #56`_: pyproject.toml: include subpackages in setuptools config
- `Issue #57`_: `!` doesn't exclude files in directories if the pattern doesn't have a trailing slash.

Improvements:

- Support Python 3.10, 3.11.
- Modernize code to Python 3.7.
- `Issue #52`_: match_files() is not a pure generator function, and it impacts tree_*() gravely.


.. _`python-path-specification`: https://github.com/cpburnz/python-path-specification
.. _`python-pathspec`: https://github.com/cpburnz/python-pathspec
.. _`pyproject.toml`: https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/
.. _`setuptools.build_meta`: https://setuptools.pypa.io/en/latest/build_meta.html
.. _`Issue #19`: https://github.com/cpburnz/python-pathspec/issues/19
.. _`Issue #35`: https://github.com/cpburnz/python-pathspec/issues/35
.. _`Issue #41`: https://github.com/cpburnz/python-pathspec/issues/41
.. _`Issue #47`: https://github.com/cpburnz/python-pathspec/issues/47
.. _`Issue #51`: https://github.com/cpburnz/python-pathspec/pull/51
.. _`Issue #52`: https://github.com/cpburnz/python-pathspec/issues/52
.. _`Issue #53`: https://github.com/cpburnz/python-pathspec/issues/53
.. _`Issue #54`: https://github.com/cpburnz/python-pathspec/issues/54
.. _`Issue #55`: https://github.com/cpburnz/python-pathspec/issues/55
.. _`Issue #56`: https://github.com/cpburnz/python-pathspec/pull/56
.. _`Issue #57`: https://github.com/cpburnz/python-pathspec/issues/57


0.9.0 (2021-07-17)
------------------

- `Issue #44`_/`Issue #50`_: Raise `GitWildMatchPatternError` for invalid git patterns.
- `Issue #45`_: Fix for duplicate leading double-asterisk, and edge cases.
- `Issue #46`_: Fix matching absolute paths.
- API change: `util.normalize_files()` now returns a `Dict[str, List[pathlike]]` instead of a `Dict[str, pathlike]`.
- Added type hinting.

.. _`Issue #44`: https://github.com/cpburnz/python-pathspec/issues/44
.. _`Issue #45`: https://github.com/cpburnz/python-pathspec/pull/45
.. _`Issue #46`: https://github.com/cpburnz/python-pathspec/issues/46
.. _`Issue #50`: https://github.com/cpburnz/python-pathspec/pull/50


0.8.1 (2020-11-07)
------------------

- `Issue #43`_: Add support for addition operator.

.. _`Issue #43`: https://github.com/cpburnz/python-pathspec/pull/43


0.8.0 (2020-04-09)
------------------

- `Issue #30`_: Expose what patterns matched paths. Added `util.detailed_match_files()`.
- `Issue #31`_: `match_tree()` doesn't return symlinks.
- `Issue #34`_: Support `pathlib.Path`\ s.
- Add `PathSpec.match_tree_entries` and `util.iter_tree_entries()` to support directories and symlinks.
- API change: `match_tree()` has been renamed to `match_tree_files()`. The old name `match_tree()` is still available as an alias.
- API change: `match_tree_files()` now returns symlinks. This is a bug fix but it will change the returned results.

.. _`Issue #30`: https://github.com/cpburnz/python-pathspec/issues/30
.. _`Issue #31`: https://github.com/cpburnz/python-pathspec/issues/31
.. _`Issue #34`: https://github.com/cpburnz/python-pathspec/issues/34


0.7.0 (2019-12-27)
------------------

- `Issue #28`_: Add support for Python 3.8, and drop Python 3.4.
- `Issue #29`_: Publish bdist wheel.

.. _`Issue #28`: https://github.com/cpburnz/python-pathspec/pull/28
.. _`Issue #29`: https://github.com/cpburnz/python-pathspec/pull/29


0.6.0 (2019-10-03)
------------------

- `Issue #24`_: Drop support for Python 2.6, 3.2, and 3.3.
- `Issue #25`_: Update README.rst.
- `Issue #26`_: Method to escape gitwildmatch.

.. _`Issue #24`: https://github.com/cpburnz/python-pathspec/pull/24
.. _`Issue #25`: https://github.com/cpburnz/python-pathspec/pull/25
.. _`Issue #26`: https://github.com/cpburnz/python-pathspec/pull/26


0.5.9 (2018-09-15)
------------------

- Fixed file system error handling.


0.5.8 (2018-09-15)
------------------

- Improved type checking.
- Created scripts to test Python 2.6 because Tox removed support for it.
- Improved byte string handling in Python 3.
- `Issue #22`_: Handle dangling symlinks.

.. _`Issue #22`: https://github.com/cpburnz/python-pathspec/issues/22


0.5.7 (2018-08-14)
------------------

- `Issue #21`_: Fix collections deprecation warning.

.. _`Issue #21`: https://github.com/cpburnz/python-pathspec/issues/21


0.5.6 (2018-04-06)
------------------

- Improved unit tests.
- Improved type checking.
- `Issue #20`_: Support current directory prefix.

.. _`Issue #20`: https://github.com/cpburnz/python-pathspec/issues/20


0.5.5 (2017-09-09)
------------------

- Add documentation link to README.


0.5.4 (2017-09-09)
------------------

- `Issue #17`_: Add link to Ruby implementation of *pathspec*.
- Add sphinx documentation.

.. _`Issue #17`: https://github.com/cpburnz/python-pathspec/pull/17


0.5.3 (2017-07-01)
------------------

- `Issue #14`_: Fix byte strings for Python 3.
- `Issue #15`_: Include "LICENSE" in source package.
- `Issue #16`_: Support Python 2.6.

.. _`Issue #14`: https://github.com/cpburnz/python-pathspec/issues/14
.. _`Issue #15`: https://github.com/cpburnz/python-pathspec/pull/15
.. _`Issue #16`: https://github.com/cpburnz/python-pathspec/issues/16


0.5.2 (2017-04-04)
------------------

- Fixed change log.


0.5.1 (2017-04-04)
------------------

- `Issue #13`_: Add equality methods to `PathSpec` and `RegexPattern`.

.. _`Issue #13`: https://github.com/cpburnz/python-pathspec/pull/13


0.5.0 (2016-08-22)
------------------

- `Issue #12`_: Add `PathSpec.match_file()`.
- Renamed `gitignore.GitIgnorePattern` to `patterns.gitwildmatch.GitWildMatchPattern`.
- Deprecated `gitignore.GitIgnorePattern`.

.. _`Issue #12`: https://github.com/cpburnz/python-pathspec/issues/12


0.4.0 (2016-07-15)
------------------

- `Issue #11`_: Support converting patterns into regular expressions without compiling them.
- API change: Subclasses of `RegexPattern` should implement `pattern_to_regex()`.

.. _`Issue #11`: https://github.com/cpburnz/python-pathspec/issues/11


0.3.4 (2015-08-24)
------------------

- `Issue #7`_: Fixed non-recursive links.
- `Issue #8`_: Fixed edge cases in gitignore patterns.
- `Issue #9`_: Fixed minor usage documentation.
- Fixed recursion detection.
- Fixed trivial incompatibility with Python 3.2.

.. _`Issue #7`: https://github.com/cpburnz/python-pathspec/pull/7
.. _`Issue #8`: https://github.com/cpburnz/python-pathspec/pull/8
.. _`Issue #9`: https://github.com/cpburnz/python-pathspec/pull/9


0.3.3 (2014-11-21)
------------------

- Improved documentation.


0.3.2 (2014-11-08)
------------------

- `Issue #5`_: Use tox for testing.
- `Issue #6`_: Fixed matching Windows paths.
- Improved documentation.
- API change: `spec.match_tree()` and `spec.match_files()` now return iterators instead of sets.

.. _`Issue #5`: https://github.com/cpburnz/python-pathspec/pull/5
.. _`Issue #6`: https://github.com/cpburnz/python-pathspec/issues/6


0.3.1 (2014-09-17)
------------------

- Updated README.


0.3.0 (2014-09-17)
------------------

- `Issue #3`_: Fixed trailing slash in gitignore patterns.
- `Issue #4`_: Fixed test for trailing slash in gitignore patterns.
- Added registered patterns.

.. _`Issue #3`: https://github.com/cpburnz/python-pathspec/pull/3
.. _`Issue #4`: https://github.com/cpburnz/python-pathspec/pull/4


0.2.2 (2013-12-17)
------------------

- Fixed setup.py.


0.2.1 (2013-12-17)
------------------

- Added tests.
- Fixed comment gitignore patterns.
- Fixed relative path gitignore patterns.


0.2.0 (2013-12-07)
------------------

- Initial release.
