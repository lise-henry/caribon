ChangeLog
=========

0.8.1 (2017-03-04)
----------------------
* Update `strsim` dependency to 0.6.0

0.8.0 (2017-01-07)
----------------------
* Add `ast_to_ispell` method to parser to allow Caribon to be runned
pretending to be ispell.
* Add `ast_to_repetitions` method to parser to make Caribon more usable as a library.

0.7.4 (2016-10-26)
----------------------
* Use `strsim` instead of `edit_distance` crate; benchmark shows that
  it increases the performances for fuzzy matching.

0.7.3 (2016-09-30)
------------------
* Use only the first two words to detect the ignored list to use, so
  it works if the user uses a unicode language code
* Add some words to french ignored list


0.7.2 (2016-08-17)
------------------
* Fix a formatting bug when printing HTML from a text input
* Runned `clippy` on repository and fixed some unesthetic stuff

0.7.1 (2015-12-10)
------------------
* Fixed documentation, homepage and repository links because of change
  of username

0.7.0 (2015-11-12)
------------------
* In `ast_to_*`, use `string.push_str(...)` instead of `string =
  string + ...`
* Replaced `to_string()` by `to_owned()` to convert `&str` to `String`
* Moved to version 0.3 of stemmer library, requiring a bit more
functions to be `&mut self`
* Fixed `Cargo.toml` to no longer use `*` dependencies which broke
  everything (sorry -_-)


0.6.2 (2015-07-15)
------------------
* Moved library files and binary files into separate directories
* Changed license to GNU Lesser General Public License
* Binary: added `--print-stats` option (a bit limited at the moment)

0.6.1 (2015-07-13)
------------------
* `Parser.detect_global` now only does something if there is enough
  words in the file compared to the threshold

0.6.0 (2015-07-08)
------------------
* Added `Parser.with_more_ignored` to add ignored words instead of replacing them
* Added `--add-ignored=` option for the binary
* Caribon now supports working on full HTML files, not just HTML
  fragments. This necessitated some changes in the API:
   * Use a struct `Ast` instead of `Vec<Word>`
   * Renamed `words_to_*` to `ast_to_*`
   * `detect_[local|global]` methods now takes a mutable ref to an Ast
   instead of moving a `Vec<Word>` and returning a new one
* Removed `detect_leak` algorithm
* Added special "no_stemmer" option to `Parser::new`: disables
stemming
* Fixed a bug in fuzzy string matching



0.5.2 (2015-07-03)
------------------
* Added `Parser.words_to_terminal` that outputs repetitions for the
terminal instead of using HTML, using terminal colours characters
* Added `Parser.words_to_markdown` that outputs repetitions in
markdown
* Added fuzzy string matching possibility, enabled by constructing the
parser with the `with_fuzzy` method
* Reworked options for the binary:
  * Added `--fuzzy=` to trigger fuzzy string matching
  * Disabled global detection by default
  * Added `--input-format` and `--output-format` options
  * Without these options, auto-detect input and output formats based
    on file extensions

0.5.1 (2015-07-01)
------------------
* Bugfixes in `Parser.tokenize` and `Parser.words_to_html` when
  handling non-html input text

0.5.0 (2015-07-01)
------------------
* Improved documentation
* `Parser.tokenize` now returns a `Result`, giving an `Error` instead of
panicking
* `Parser::new` also returns a `Result` instead of an `Option`
* `detect_leak` now uses `max_distance` to compute a leak, so `leak`
parameter is removed.
* Therefore, `--leak=x` argument is removed from the binary
* Options for the binary have some more changes (see `--help`)
* It is now possible to chain detection methods, highlighting
with different colours
* Added `words_stats` method



0.4.0 (2015-06-30)
------------------
* `words_to_html` is now part of Parser implementation
* If `parser.html` is false (input type is not html),
  `parser.words_to_html` convert line breaks to `<br/>`
* Escape characters such as `&nbsp;` are leaved as such if
  `parser.html` is true
* Better support for user-defined list of ignored words in the library
* New option `--ignore=` in the binary
  

0.3.0 (2015-06-29)
------------------
* `words_to_html` takes another argument, `standalone`
* removed "--debug" option
* added "--input" and "--output" options

0.2.2 (2015-06-28)
------------------
* Fixed a bug (and unnecessary clone) in detect_local algorithm
* Added a list of ignored words for english

0.2.1 (2015-06-28)
------------------
* Added documentation for the library and examples on Github.io
* Minor bugfixes

0.2.0 (2015-06-28)
------------------
* Added support for HTML in input
* Added some ignored words for french
* Added option to ignore proper nouns
* Added `detect_local` algorithm
* Added `Parser::list_languages()` function
* Added support for command-line arguments

0.1.0 (2015-06-27)
------------------

* first release
